# Bacon + Gravity Forms library

'Reset' CSS library for use with [Bacon](https://github.com/netzstrategen/bacon) and WordPress Gravity Forms plugin.

## Why?

Gravity Forms includes opinionated styling by default which must nearly always be overridden to match the project form styling. In addition, this Gravity Forms CSS is of poor quality and requires highly specific selectors to override.

The goal of this library is to 'reset' Gravity Forms so that only the required layout styling from Gravity Forms is applied. Form element styling will then be applied automatically by Bacon (which is a hard dependency of this library).

## Installation and usage
- Ensure your project is using [Bacon](https://github.com/netzstrategen/bacon)
- `npm i @netzstrategen/bacon-gravityforms`
- Include via `@import '~@netzstrategen/bacon-gravityforms'` directly after Bacon (note that this syntax is from [`sass-module-importer`](https://github.com/lucasmotta/sass-module-importer) which is included in most netzstrategen projects via [`gulp-task-collection`](https://github.com/netzstrategen/gulp-task-collection))

### Required changes in PHP
This library requires some minor markup changes which can be done via Gravity Forms filters. Include the following class in the WordPress install (preferably in somewhere like `theme/src/GravityForms.php`). This code will:
- Dequeue Gravity Forms default stylesheets
- Add a hookable class to the wrapping `<form>` tag
- Change checkbox and radio button markup to match that required by Bacon

```
<?php

namespace xxx/xxx;

/**
 * Adjust Gravity Forms plugin functionality.
 */
class GravityForms {

  /**
   * Plugin initialization method.
   */
  public static function init() {
    add_filter('pre_option_rg_gforms_disable_css', '__return_true');
    add_filter('gform_form_tag', __CLASS__ . '::adjustFormWrapper', 10, 2);
    add_filter('gform_field_choice_markup_pre_render', __CLASS__ . '::adjustChoiceInputs', 10, 4);
  }

  /**
   * Adds theme wrapper classes to Gravity Form wrapper element.
   *
   * @implements adjustFormWrapper
   */
  public static function adjustFormWrapper($form_tag, $form) {
    $form_tag = str_replace("<form", "<form class='gform_form_tag'", $form_tag);
    return $form_tag;
  }

  /**
   * Adjust choice inputs markup to match Bacon.
   *
   * @implements adjustChoiceInputs
   */
  public static function adjustChoiceInputs($choice_markup, $choice, $field, $value) {
    if ($field->get_input_type() == 'radio' || $field->get_input_type() == 'checkbox') {

      // Get wrapping <li>, input element and label text.
      preg_match('@<li[^>]*>@', $choice_markup, $wrapper_matches);
      preg_match('@<input[^>]*>@', $choice_markup, $input_matches);
      preg_match('@<label.*?>(.*)<\/label>@', $choice_markup, $label_matches);

      if (!empty($wrapper_matches) && !empty($input_matches) && !empty($label_matches)) {
        // Get wrapping <li> element.
        $opening_wrapper = $wrapper_matches[0];

        // Get label text.
        $label = $label_matches[1];

        // Get entire <input> element.
        $input = $input_matches[0];

        // Build new Bacon markup.
        $new_markup = $opening_wrapper;
        $new_markup .= '<label class="choice-input ' . $field->get_input_type() . '">';
        $new_markup .= $input;
        $new_markup .= '<span class="choice-input__indicator ' . $field->get_input_type() . '__indicator"></span>';
        $new_markup .= '<span class="choice-input__label">' . $label . '</span>';
        $new_markup .= '</label>';
        $new_markup .= '</li>';

        return $new_markup;
      }
    }

    return $choice_markup;
  }

}
```

## Troubleshooting

It is known that not all layout CSS has been moved into this library from the Gravity Forms stylesheet.

It is also known that this library may require updating from time-to-time dependent on changes made to the Gravity Forms plugin.

If you encounter missing or incomplete layout CSS then first check the primary Gravity Forms stylesheet (located at `plugins/gravityforms/css/formsmain.css`) and secondary stylesheets and copy any relevant CSS into the `Base Gravity Forms fields layout` section of [_bacon-gravityforms.scss_](https://github.com/netzstrategen/bacon-gravityforms/blob/master/_bacon-gravityforms.scss).
