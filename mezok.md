
A WooCommerce alapértelmezetten 6 típusú egyedi mezőt támogat úgynevezett built in módon. 
* Text
* Textarea
* Number
* Checkbox
* Radio
* Select (dropdown)

A mezők létrehozásakor úgynvezett attributumok segítségével tudjuk meghatározni az egyes tulajdonságokat.

**Példa:**
```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_text_field' );
function add_custom_text_field() {
    woocommerce_wp_text_input( array(
        'id'          => '_custom_product_text_field', 
        'label'       => __('Custom Text Field', 'woocommerce'), 
        'placeholder' => __('Enter text here', 'woocommerce'),
        'desc_tip'    => true, 
        'description' => __('This is a custom text field.', 'woocommerce'), 
        'type'        => 'text',
        'default'     => '', 
    ));
}
```

ID             -> Ez lesz a meta azonosítója, nem tartalmazhat speciális karaktereket vagy ékezetet. Minden meződnek eltérő azonosítóval kell rendelkeznie.
LABEL          -> Ez lesz a mező címkéje, neve ami megjelenik az egyedi mező felett.
PLACEHOLDER    -> Másnéven helykitöltő, egyes mezők esetében ez jelenik meg az input-ban. (például a text, vagy textarea esetében)
DESC_TIP       -> A WooCommerce beépített Tooltip kezelője, ha az érték true, azaz igaz, a *description* azaz leírás tooltip ként fog megjelenni.
