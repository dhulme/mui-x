---
productId: x-tree-view
title: Rich Tree View - Headless
packageName: '@mui/x-tree-view'
githubLabel: 'component: tree view'
waiAria: https://www.w3.org/WAI/ARIA/apg/patterns/treeview/
---

# Rich Tree View - Headless

<p class="description">Create your custom tree view.</p>

:::warning
The `useTreeView` hook is not public API for now,
this page is still a _Work in Progress_ and its content is not meant to be used in production.
:::

## Create a custom plugin

### Basic plugin

A custom plugins contains 2 required elements:

1. A hook that will be executed by the `useTreeView` hook:

   ```ts
   const useCustomPlugin = ({ params }) => {
     React.useEffect(() => {
       console.log(params.customParam);
     });
   };
   ```

2. A property containing the params used by this topic:

   ```ts
   useCustomPlugin.params = {
     customParam: true,
   };
   ```

### Params default value

You can use the `getDefaultizedParams` property to set a default value to your plugin params:

```ts
const useCustomPlugin = ({ params }) => {
  React.useEffect(() => {
    console.log(params.customParam);
  });
};

useCustomPlugin.params = { customParam: true };

useCustomPlugin.getDefaultizedParams = (params) => ({
  ...params,
  customParam: params.customParam ?? false,
});
```

### Controllable models

A model is a value that can either be controlled or initialized using a prop.
The Tree View contains several models like the `expandedNodes` model which contains the ids of the nodes currently expanded.

You can create your own models using the `models` property of your plugin:

```ts
useCustomPlugin.params = {
  // ...other params
  defaultCustomModel: true,
  customModel: true,
};

useCustomPlugin.getDefaultizedParams = (params) => ({
  ...params,
  // ... other defaultized params
  defaultCustomModel: params.defaultCustomModel ?? false,
});

useCustomPlugin.models = {
  customModel: {
    getDefaultValue: (params) => defaultCustomModel,
  },
};
```

:::info
When creating a model, you should always set a default value to your `defaultCustomModel` prop. This will initialize the component's internal state when the model is uncontrolled.
See the [Params default value](/x/react-tree-view/rich-tree-view/headless/#params-default-value) section for more information.
:::

You can then use this model in your plugin (or in any other plugin) using the `models` parameter:

```ts
const useCustomPlugin = ({ models }) => {
  React.useEffect(() => {
    console.log(models.customModel.value);
  });

  const updateCustomModel = (newValue) =>
    models.customModel.setControlledValue(newValue);
};
```

### Use elements from another plugin

Your plugin can access the instance methods, the params and the models of any other plugin.

```ts
const useCustomPlugin = ({ models }) => {
  const handleSomeAction = () => {
    // Log the id of the nodes currently expanded
    console.log(models.expandedNodes.value);

    // Collapse all the nodes
    models.expandedNodes.setValue([]);

    // Check if a node is expanded
    const isExpanded = instance.isNodeExpanded('some-node-id');
  };
};
```

:::warning
Setting a model value in another plugin is wrong because it won't publish the associated callback.
We probably need a new abstraction here so that a plugin is always responsible for its model updates.
:::

### Define a plugin state

TODO

:::warning
Once `focusedNodeId` becomes a model, we could consider removing the notion of state and just let each plugin define its state and provide methods in the instance to access / update it.
:::

### Populate the Tree View instance

The Tree View instance is an object accessible in all the plugins and in the `TreeItem`.
It is the main way a plugin can provide features to the rest of the component.

```ts
const useCustomPlugin = ({ models, instance }) => {
  const toggleCustomModel = () =>
    models.customModel.setValue(!models.customModel.value);

  populateInstance(instance, { toggleCustomModel });
};
```

You can then use this instance method in any other plugin:

```ts
const useOtherCustomPlugin = ({ models, instance }) => {
  const handleSomeAction = () => {
    instance.toggleCustomModel();
  };
};
```

### Emit and receive events

```ts
const useCustomPlugin = () => {
  const toggleCustomModel = () => {
    const newValue = !models.customModel.value;
    models.customModel.setValue(newValue);
    publishTreeViewEvent(instance, 'toggleCustomModel', { value: newValue });
  };

  populateInstance(instance, { toggleCustomModel });
};
```

You can then subscribe to this event in any other plugin:

```ts
const useOtherCustomPlugin = ({ instance }) => {
  useInstanceEventHandler(instance, 'toggleCustomModel', ({ value }) => {
    console.log('New value of customModel', value);
  });
};
```

:::info
If you are using TypeScript, you need to define your dependencies in your plugin signature, see the [Plugin typing](/x/react-tree-view/rich-tree-view/headless/#plugin-typing) section for more information.
:::

### Pass props to your root element

You can use the `getRootProps` property of your returned value to pass props to your root element:

```ts
const useCustomPlugin = ({ params }) => {
  return {
    getRootProps: () => ({
      'data-customparam': params.customParam,
    }),
  };
};
```

### Plugin typing

The typing of a plugin is defined using its _signature_.
This type contains the following information:

```ts
type UseCustomPluginSignature = TreeViewPluginSignature<{
  // The params specific to your plugin before running `getDefaultizedParams`
  params: UseCustomPluginParams;
  // The params specific to your plugins after running `getDefaultizedParams`
  defaultizedParams: UseCustomPluginDefaultizedParams;
  // The methods added to the tree view instance by your plugin
  instance: UseCustomPluginInstance;
  // The events emitted by your plugin
  events: UseCustomPluginEvents;
  // The states defined by your plugin
  state: UseCustomPluginState;
  // The context value defined by your plugin and passed to the items
  contextValue: UseCustomPluginContextValue;
  // The name of the models defined by your plugin
  modelNames: UseCustomPluginModelNames;
  // The plugins this plugin needs to work correctly
  dependantPlugins: UseCustomPluginDependantPlugins;
}>;
```

The most basic plugin would have the following signature:

```ts
type UseCustomPluginSignature = TreeViewPluginSignature<{}>;
```

The plugin built in the sections above would have the following signature:

```ts
type UseCustomPluginSignature = TreeViewPluginSignature<{
  params: {
    customParam?: boolean;
    customModel?: boolean;
    defaultCustomModel?: boolean;
  };
  // `customParam` and `defaultCustomModel` have a default value defined in `getDefaultizedParams`
  defaultizedParams: {
    customParam: boolean;
    customModel?: boolean;
    defaultCustomModel: boolean;
  };
  instance: { toggleCustomModel: () => void };
  events: {
    toggleCustomModel: {
      params: { value: boolean };
    };
  };
  modelNames: 'customModel';
  // We want to have access to the expansion models and methods of the expansion plugin.
  dependantPlugins: [UseTreeViewExpansionSignature];
}>;
```

## Examples

### Log expanded items

Interact with the tree view to see the expanded items being logged:

{{"demo": "LogExpandedNodes.js"}}
