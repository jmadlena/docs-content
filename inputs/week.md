# Week input

The `week` input uses the HTML's [native week picker](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/week) to allow users to easily
select a week.

<callout type="danger" label="Compatibility warning">
Month inputs are not currently supported in FireFox or Safari.
</callout>

<example
  name="Week input"
  file="/_content/examples/week/week"
  langs="vue">
</example>

<callout type="warning" label="Formatting">
The internal format of all native week pickers is <code>YYYY-Www</code> (for example: <code>2017-W06</code>). This is true even though the format of the week displayed to the user may be different.</code>.
</callout>

## Props & Attributes

The `week` input has no unique props but can make use of the following universal
FormKit props.

<reference-table input="week" :attrs="['min', 'max', 'step']">
</reference-table>

## Composition keys

<reference-table type="compositionKeys" primary="composition-key">
</reference-table>

## Available utilities

[TK] - Masks

[TK] - Casts