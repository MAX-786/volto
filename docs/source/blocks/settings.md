---
myst:
  html_meta:
    "description": "How to configure custom blocks"
    "property=og:description": "How to configure custom blocks"
    "property=og:title": "Blocks settings"
    "keywords": "Volto, Plone, frontend, React, Block settings"
---

# Blocks settings

Volto has a set of default blocks.
You can extend it by adding your custom blocks in your project's policy add-on configuration object.

## Configuring a new block

To extend the default set of blocks, you add the following lines to a file like `frontend/packages/volto-project-title/src/index.js`.
`frontend/packages/volto-project-title` is your policy package where you configure your project and make customizations.
Adapt the name `volto-project-title` according to your needs.

```js
import MainSliderViewBlock from './components/Blocks/MainSlider/View';
import MainSliderEditBlock from './components/Blocks/MainSlider/Edit';
import sliderSVG from '@plone/volto/icons/slider.svg';

import SimpleTeaserView from './components/Blocks/SimpleTeaserView';
import CardTeaserView from './components/Blocks/CardTeaserView';
import DefaultColumnRenderer from './components/Blocks/DefaultColumnRenderer';
import NumberColumnRenderer from './components/Blocks/NumberColumnRenderer';
import ColoredColumnRenderer from './components/Blocks/ColoredColumnRenderer';

import CustomSchemaEnhancer from './components/Blocks/CustomSchemaEnhancer';


const applyConfig = (config) => {
  [...]

  // Register new block for homepage teaser
  config.blocks.blocksConfig.mainslider = {
    id: 'mainslider', // The name (id) of the block
    title: 'Main Slider', // The display name of the block
    icon: sliderSVG, // The icon used in the block chooser
    group: 'common', // The group (blocks can be grouped, displayed in the chooser) ['common', 'text', 'teasers', 'media']
    view: MainSliderViewBlock, // The view mode component
    edit: MainSliderEditBlock, // The edit mode component
    restricted: false, // {Boolean|function} If the block is restricted, it won't show in the chooser. The function signature is `({properties, block, navRoot, contentType})` where `properties` is the current object data and `block` is the block being evaluated in `BlockChooser`. `navRoot` is the nearest navigation root object and `contentType` is the current content type.
    mostUsed: true, // A meta group `most used`, appearing at the top of the chooser
    blockHasOwnFocusManagement: false, // Set this to true if the block manages its own focus
    sidebarTab: 1, // The sidebar tab you want to be selected when selecting the block
    blockHasValue: (data) => {
      // Returns true if the provided block data represents a value for the current block.
      // Required for alternate default block types implementations.
      // See also [Settings reference](/configuration/settings-reference)
    },
    // The `blockSchema` property can either be a schema by itself
    // (a JavaScript object describing the schema),
    // or a function that returns a schema.
    blockSchema: CustomSchema,
    // A block can have an schema enhancer function with the signature: (schema) => schema
    // It can be either be at block level (it's applied always), at a variation level
    // or both. It's up to the developer to make them work nicely (not conflict) between them
    schemaEnhancer: CustomSchemaEnhancer,
    // A block can define variations (it should include the stock, default one)
    variations: [
      {
        id: 'default',
        title: 'Default',
        isDefault: true,
        render: SimpleTeaserView
      },
      {
        id: 'card',
        label: 'Card',
        render: CardTeaserView,
        schemaEnhancer: ({schema, formData, intl}) => {
          schema.properties.cardSize = '...'; // fill in your implementation
          return schema;
        }
      }
    ],
    // A block can define extensions that enhance the default stock block behavior
    extensions: {
      columnRenderers: {
        title: messages.title,
        items: [
          {
            id: 'default',
            title: 'Default',
            isDefault: true,
            render: DefaultColumnRenderer
          },
          {
            id: 'number',
            title: 'Number',
            render: NumberColumnRenderer,
          },
          {
            id: 'colored',
            title: 'Colored',
            renderer: ColoredColumnRenderer,
            schemaEnhancer: ({formData, schema, intl}) => {
              schema.properties.color = {
                widget: 'color',
                title: 'Color',
              };
              schema.fieldsets[0].fields.push('color');
              return schema;
            }
          }
        ]
      }
    }
  };

  [...]

  return config;
};

export default applyConfig;
```

We start by importing both view and edit components of our recently created custom block.

Then you define the block, using the object described in the example.

We also add this piece of code in order to define i18n literals for our new block:

```js
import { defineMessages } from 'react-intl';

...

defineMessages({
  mainslider: {
    id: 'Main Slider',
    defaultMessage: 'Main Slider',
  },
});
```

Our new block should be ready to use in the editor by selecting it in the editor's block chooser.

## Common block options

It is a common pattern to use the block configuration to allow customization of a block's behavior or to provide block-specific implementation of various Volto mechanisms.
Some of these common options are described in the following sections.

(blockHasValue)=

### `blockHasValue`

`blockHasValue` is a function that returns `true` if the provided block data represents a non-empty value for the current block.
Required for alternate default block types implementations.
It has the following signature.

```jsx
blockHasValue(data) => boolean
```

### `initialValue`

`initialValue` is a function that can be used to get the initial value for a block.
It has the following signature.

```jsx
initialValue({id, value, formData, intl}) => newFormData
```

### `blockSchema`

A must-have for modern Volto blocks, `blockSchema` is a function, or directly the schema object, that returns the schema for the block data.
Although it's not required, defining the schema enables the block to have its initial value based on the default values declared in the schema.

### `disableEnter`

Normally when a block is selected and you press {kbd}`enter`, a new block is inserted below.
When you don't want this behavior and want to handle the {kbd}`enter` input yourself inside the block, set `disableEnter` to `true`.

## Other block options

The configuration object also exposes these options

### requiredBlocks - The required (mandatory, cannot be removed) blocks

This option is used to make the tiles not removable. By default, the Title block is not removable (you won't be able to delete it as the remove handler is not present).

### groupBlocksOrder - The blocks chooser group order

This option is used to define the order of the groups in the blocks chooser. By default:

```js
const groupBlocksOrder = [
  { id: 'mostUsed', title: 'Most used' },
  { id: 'text', title: 'Text' },
  { id: 'media', title: 'Media' },
  { id: 'common', title: 'Common' },
];
```

You can change it (and add your own group) in your project configuration object.

### initialBlocks - Initial Blocks per content type

By default, the default blocks for all content types are a title block and a text block. You can override this and provide your own by modifying the configuration object:

```js
const initialBlocks = {};
```

and provide your own per content type, e.g:

```js
const initialBlocks = {
    Document: ['leadimage', 'title', 'text', 'listing' ]
};
```

You can also pass the full configuration for the block using an object:

```js
const initialBlocks = {
  Document: [
    { '@type': 'leadImage', fixed: true, required: true },
    { '@type': 'title' },
    { '@type': 'slate', value: 'My default text', plaintext: 'My default text' },
  ],
};
```

## Listing block configuration

`allowed_headline_tags`
: Allows you to customize the choices of the "Headline Tag" types shown in the block settings by default. It has the following syntax (a list of lists, where a list item consists of `['token', 'display_name']`):

  ```js
  allowed_headline_tags: [['h2', 'h2'], ['h3', 'h3']]
  ```

  If not specified, an internal hardcoded default is the above shown example.

  If the choice is limited to one item, then the setting hides itself from the `listing` block settings list.

## Search block configuration

The search block provides several extensibility options.

### Variations

The search block uses the variations to provide alternate layout.

### FacetWidgets rewriteOptions extension

Sometimes the labels provided by a field are not directly usable in UI. You can
override the `rewriteOptions` function. Don't be alarmed by the facet that's
already filled in to handle `review_state`. You can save a reference to the
current function and define a new function that handles another field, that
also calls the old saved function.

### FacetWidgets types

This allows definition of new facet filtering widgets. In addition to the
`view` field, which defines the component to be used to render the facet
widget, you need to also set:

- `schemaEnhancer`: a schema extender for the object list widget that's used to
  edit each facet setting
- `stateToValue`: a function to convert the state (extracted from URL) to
  a value that can be used in the facet widget
- `valueToQuery`: a function that converts the value of the widget to state,
  something that can be used to compose the querystring query
- `filterListComponent`: component to be used in the filter list display of
  that facet.
