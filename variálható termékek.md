Variálható termékek esetében némileg eltérő módon kell létrehoznunk az egyes egyedi mezőket.

```
add_action( 'woocommerce_product_after_variable_attributes', 'add_custom_field_to_variations', 10, 3 );
function add_custom_field_to_variations( $loop, $variation_data, $variation ) {
    woocommerce_wp_text_input( array(
        'id'            => '_custom_variation_text_field_' . $loop,
        'label'         => __( 'Egyedi Szövegmező', 'woocommerce' ),
        'desc_tip'      => 'true',
        'description'   => __( 'Adj meg egy egyedi szöveget a variációnak.', 'woocommerce' ),
        'value'         => get_post_meta( $variation->ID, '_custom_variation_text_field', true ),
        'wrapper_class' => 'form-row form-row-full',
    ));
}

add_action( 'woocommerce_save_product_variation', 'save_custom_field_variations', 10, 2 );
function save_custom_field_variations( $variation_id, $i ) {
    $custom_field = isset( $_POST['_custom_variation_text_field_' . $i] ) ? sanitize_text_field( $_POST['_custom_variation_text_field_' . $i] ) : '';
    update_post_meta( $variation_id, '_custom_variation_text_field', $custom_field );
}
```

Számos attributum azonos a korábbiakhoz képest, a két fontos eltérés a *value* valamint *wrapper_class* esetében látható. A value a mező aktuális értékét töltjük be, lévén hogy minden variációnak lehet saját értéke. A wrapper class esetében a WooCommerce WordPress alapértelmezett osztályait definiáljuk,
így a megjelenítés konzisztens. A *form-row-full* biztosítja, hogy a meta mezőnk 100% széles legyen. A *form-row-first* biztosítod hogy az adott metád oszloposan az "első oszlopban legyen".

Nézzünk meg, hogyan lehetne a mező értékét a frontend is megjeleníteni. Ez némileg több lépést igényel, ugyanis az interakció miatt, javascript kiegészítésre lesz szükségünk.

```
add_action( 'woocommerce_product_after_variable_attributes', 'add_custom_field_to_variations', 10, 3 );
function add_custom_field_to_variations( $loop, $variation_data, $variation ) {
    woocommerce_wp_text_input( array(
        'id'            => '_custom_variation_text_field_' . $loop,
        'label'         => __( 'Egyedi Szövegmező', 'woocommerce' ),
        'desc_tip'      => 'true',
        'description'   => __( 'Adj meg egy egyedi szöveget a variációnak.', 'woocommerce' ),
        'value'         => get_post_meta( $variation->ID, '_custom_variation_text_field', true ),
        'wrapper_class' => 'form-row form-row-first',
    ));
}

add_action( 'woocommerce_save_product_variation', 'save_custom_field_variations', 10, 2 );
function save_custom_field_variations( $variation_id, $i ) {
    $custom_field = isset( $_POST['_custom_variation_text_field_' . $i] ) ? sanitize_text_field( $_POST['_custom_variation_text_field_' . $i] ) : '';
    update_post_meta( $variation_id, '_custom_variation_text_field', $custom_field );
}

add_filter( 'woocommerce_available_variation', 'add_custom_field_to_variation_data', 10, 3 );
function add_custom_field_to_variation_data( $variation_data, $product, $variation ) {
    $custom_field_value = get_post_meta( $variation->get_id(), '_custom_variation_text_field', true );
    $variation_data['custom_field'] = $custom_field_value;
    return $variation_data;
}

add_action( 'wp_footer', 'custom_variation_field_display_script' );
function custom_variation_field_display_script() {
    if ( !is_product() ) return;  // Csak termékoldalakon

    ?>
    <script type="text/javascript">
		jQuery(document).ready(function($) {
			// Figyeljük a variáció kiválasztását
			$('form.variations_form').on('show_variation', function(event, variation) {
		$('.custom-variation-field').remove(); // Eltávolítja az előző mezőt

		if (variation.custom_field) {
			var customFieldContent = $('<div/>', {
				'class': 'custom-variation-field',
				'text': variation.custom_field
			});

			$('.single_variation').before(customFieldContent);
		}
	}).on('hide_variation', function(event) {
		$('.custom-variation-field').remove(); // Eltávolítja a mezőt, ha nincs variáció kiválasztva
	});
    });
    </script>
    <?php
}
```

A fenti kód egy teljes példa, emiben az egyes variálható termékekhez hozzáadtunk egy text mezőt, és kezeljük annak mentését. A kód második szakasza tartalmaz egy adathozzáférést, *woocommerce_available_variation* segítségével a javascript (pontosabban jQuery) által átadhatóak az adatok, amiket így megjeleníthetünk a frontend oldalon. A scriptet a footerben helyeztük el, és szabályozzuk, hogy csak a termék single oldalon fut. A js reagál a felhasználó által kiválasztott variációkra, és amennyiben van megjeleníthető egyedi adat megjeleníti azt a kosárhoz adás felett. 

Mivel a variálható termékek esetében az egyedi mező hozzáadás némileg eltér a normál egyszerű termékek esetében, nézzünk meg egy további példát, ahol egy speciális multi select dropdown mezőt adunk hozzá. Az újabb kód a backend és frontend kezelést egyaránt kezelni fogja a fentihez hasonlóan.

```
	add_action( 'woocommerce_product_after_variable_attributes', 'add_custom_multiple_select_to_variations', 10, 3 );
function add_custom_multiple_select_to_variations( $loop, $variation_data, $variation ) {
    // A választható opciók
    $options = array(
        'option1' => 'Opció 1',
        'option2' => 'Opció 2',
        'option3' => 'Opció 3',
        // További opciók
    );

    // Az aktuális értékek beolvasása
    $current_values = get_post_meta( $variation->ID, '_custom_variation_multiple_select', true );
    $current_values = !empty($current_values) ? explode(',', $current_values) : array();

    // Mező megjelenítése
    echo '<div class="form-row form-row-full">';
    echo '<label>' . __( 'Többszörös Választás', 'woocommerce' ) . '</label>';
    echo '<select multiple name="_custom_variation_multiple_select_' . $loop . '[]" class="wc-enhanced-select">';
    foreach ($options as $value => $label) {
        echo '<option value="' . esc_attr($value) . '" ' . (in_array($value, $current_values) ? 'selected' : '') . '>' . esc_html($label) . '</option>';
    }
    echo '</select>';
    echo '</div>';
}

// mentés

add_action( 'woocommerce_save_product_variation', 'save_custom_multiple_select_variations', 10, 2 );
function save_custom_multiple_select_variations( $variation_id, $i ) {
    $custom_field = isset( $_POST['_custom_variation_multiple_select_' . $i] ) ? $_POST['_custom_variation_multiple_select_' . $i] : '';
    $custom_field = is_array($custom_field) ? implode(',', array_map('sanitize_text_field', $custom_field)) : '';
    update_post_meta( $variation_id, '_custom_variation_multiple_select', $custom_field );
}

// átadás

add_filter( 'woocommerce_available_variation', 'add_custom_multiple_select_to_variation_data', 10, 3 );
function add_custom_multiple_select_to_variation_data( $variation_data, $product, $variation ) {
    $custom_field_value = get_post_meta( $variation->get_id(), '_custom_variation_multiple_select', true );
    $variation_data['custom_multiple_select'] = $custom_field_value;
    return $variation_data;
}


// megjelenítés

add_action( 'wp_footer', 'custom_multiple_select_field_display_script' );
function custom_multiple_select_field_display_script() {
    if ( !is_product() ) return;

    ?>
    <script type="text/javascript">
    jQuery(document).ready(function($) {
        $('form.variations_form').on('show_variation', function(event, variation) {
            $('.custom-multiple-select-field').remove();

            if (variation.custom_multiple_select) {
                var selectedOptions = variation.custom_multiple_select.split(',').join(', ');
                var customFieldContent = $('<div/>', {
                    'class': 'custom-multiple-select-field',
                    'text': selectedOptions
                });

                $('.single_variation').before(customFieldContent);
            }
        }).on('hide_variation', function(event) {
            $('.custom-multiple-select-field').remove();
        });
    });
    </script>
    <?php
}
```
