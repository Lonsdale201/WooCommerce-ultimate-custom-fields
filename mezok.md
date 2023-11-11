
A WooCommerce alapértelmezetten 6 típusú egyedi mezőt támogat úgynevezett built in módon. 
* Text
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
