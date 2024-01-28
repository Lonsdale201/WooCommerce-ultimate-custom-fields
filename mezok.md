
## A WooCommerce alapértelmezetten 6 típusú egyedi mezőt támogat úgynevezett built in módon. 

* Text
* Textarea
* Number
* Checkbox
* Radio
* Select (dropdown)

## Speciális mezők

* Multiple select
* Dátum választás (HTML 5)

## A mezők létrehozásakor úgynevezett attributumok segítségével tudjuk meghatározni az egyes tulajdonságokat.

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
A példa kód egy egyszerű text meta mezőt fog létrehozni, amely közvetlenül a **Álatalános** tabfülön az árak alatt jelenik meg (woocommerce_product_options_general_product_data). A poziciót, és hogy melyik tabfülbe szeretnénk elhelyezni, vagy esetleg egy teljesen újban szintén lehetséges.

* ID             -> Ez lesz a meta azonosítója, nem tartalmazhat speciális karaktereket vagy ékezetet. Minden meződnek eltérő azonosítóval kell rendelkeznie.
* LABEL          -> Ez lesz a mező címkéje, neve ami megjelenik az egyedi mező felett.
* PLACEHOLDER    -> Másnéven helykitöltő, egyes mezők esetében ez jelenik meg az input-ban. (például a text, vagy textarea esetében)
* DESC_TIP       -> A WooCommerce beépített Tooltip kezelője, ha az érték true, azaz igaz, a *description* azaz leírás tooltipként fog megjelenni.
* DESCRIPTION    -> A mező leírása, ami a mező alatt jelenik meg, vagy tooltip *true* esetében a tooltipben.
* TYPE           -> Itt határozod meg a mező típusát.
* DEFAULT        -> Legyen-e alapértelmezett érték megadva, ha nem hagy '' üresen.

Ha kész az egyedi mezőnk, gondoskodnunk kell arról, hogy ezt megfelelően kezeljük, és mentsük az adatbázisban. A mentést két részre is bonthatjuk, az alábbi példában ellenőrizzük azt, hogy üres-e a mező, Amennyiben igen eltávolítjuk a metaadatot. Ezt a megközelítést jobban 
preferálom, mert nincs szükségem üres, vagy érvénytelen értékekre.

```
add_action( 'woocommerce_process_product_meta', 'save_custom_text_field' );
function save_custom_text_field( $post_id ) {
    if ( isset( $_POST['_custom_product_text_field'] ) ) {
        $custom_field_value = sanitize_text_field( $_POST['_custom_product_text_field'] );
        
        // Ha a mező üres, töröljük a metaadatot
        if ( empty( $custom_field_value ) ) {
            delete_post_meta( $post_id, '_custom_product_text_field' );
        } else {
            // Ha van érték, frissítjük a metaadatot
            update_post_meta( $post_id, '_custom_product_text_field', $custom_field_value );
        }
    } else {
        // Ha a mező nem létezik, töröljük a metaadatot
        delete_post_meta( $post_id, '_custom_product_text_field' );
    }
}
```
### Mező értékének megjelenítése
Azonban sok esetben ezeket szeretnénk megjeleníteni a frontend-en, tehát itt még nem áll meg a dolog. Az első példában automatizáljuk, azaz teljesen függetlenül mindentől vezérelten jelenítjük meg, például a kosárhoz adás form alatt. Ha máshol szeretnéd a termék single oldlaán megjeleníteni, akkor cseréld le a *woocommerce_after_add_to_cart_form* 
```
// megjelenítés

add_action( 'woocommerce_after_add_to_cart_form', 'display_custom_field', 20 );
function display_custom_field() {
    global $post;

    // Lekérjük a saját metánk értékét
    $custom_field_value = get_post_meta( $post->ID, '_custom_product_text_field', true );

    // Ellenőrizzük, hogy van-e érték a mezőben
    if ( ! empty( $custom_field_value ) ) {
        // Érték megjelenítése itt
        echo '<div class="custom-field">';
        echo '<p>' . esc_html( $custom_field_value ) . '</p>';
        echo '</div>';
    }
}
```
Azonban előfordul hogy kicsit speciálisabb a helyzet, adott egy page builder, és sokkal jobb lenne ha inkább shortcode állna a rendelkezésünkre. Az alábbi példa Shortcode ot készít az egyedi mezőnköz, ami aztán a termék single sablonban bárhol elhelyezhetünk. a shortcode: **[display_custom_field]**
```
// megjelenítés

function display_custom_field_shortcode( $atts ) {
    $atts = shortcode_atts( array(), $atts, 'display_custom_field' );

    global $post;

    // Lekérjük az egyéni mező értékét
    $custom_field_value = get_post_meta( $post->ID, '_custom_product_text_field', true );

    if ( ! empty( $custom_field_value ) ) {
        return '<div class="custom-field">' . esc_html( $custom_field_value ) . '</div>';
    }

    // Ha nincs érték, ne jelenítsünk meg semmit
    return '';
}
add_shortcode( 'display_custom_field', 'display_custom_field_shortcode' );
```
Nézzünk meg egy utolsó opciót ami szintén a Shortcode-hoz tartozik, azonban felveszünk egy attributumot label néven. Maga a Shortcode definiála nem változik  **[display_custom_field]** de, most már kiegészíheted: **[display_custom_field label="Text field: "]**

```
// megjelenítés

function display_custom_field_shortcode( $atts ) {
    // Shortcode attribútum
    $atts = shortcode_atts( array(
        'label' => '', // Alapértelmezetten üres
    ), $atts, 'display_custom_field' );

    global $post;

    // Lekérjük az egyéni mező értékét
    $custom_field_value = get_post_meta( $post->ID, '_custom_product_text_field', true );

    if ( ! empty( $custom_field_value ) ) {
        // Label megjelenítése ha van megadva
        $label_html = '';
        if ( ! empty( $atts['label'] ) ) {
            $label_html = '<span class="custom-field-label">' . esc_html( $atts['label'] ) . '</span>';
        }

        // A label és az érték megjelenítése
        return '<div class="custom-field-wrapper">' . $label_html . '<span class="custom-field-value">' . esc_html( $custom_field_value ) . '</span></div>';
    }

    // Ha nincs érték, ne jelenítsünk meg semmit
    return '';
}
add_shortcode( 'display_custom_field', 'display_custom_field_shortcode' );
```

## Példa az összes típusú meta mezőre:

### Textarea típusú meta mező

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_textarea_field' );
function add_custom_textarea_field() {
    woocommerce_wp_textarea_input( array(
        'id'          => '_custom_product_textarea',
        'label'       => __('Custom Textarea', 'woocommerce'),
        'placeholder' => __('Enter details here', 'woocommerce'),
        'description' => __('This is a custom textarea field.', 'woocommerce'),
    ));
}
```


### Number (szám) típusú meta mező

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_number_field' );
function add_custom_number_field() {
    woocommerce_wp_text_input( array(
        'id'          => '_custom_product_number_field',
        'label'       => __('Custom Number Field', 'woocommerce'),
        'placeholder' => __('Enter a number', 'woocommerce'),
        'description' => __('This is a custom number field.', 'woocommerce'),
        'type'        => 'number',
        'custom_attributes' => array(
            'step'  => 'any',
            'min'   => '0'
        )
    ));
}
```

### Checkbox (jelölőnégyzet) típusú meta mező

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_checkbox_field' );
function add_custom_checkbox_field() {
    global $post;

    // Lekérjük a mentett értéket
    $checkbox_value = get_post_meta( $post->ID, '_custom_product_checkbox', true );

    woocommerce_wp_checkbox( array(
        'id'          => '_custom_product_checkbox',
        'label'       => __('Custom Checkbox', 'woocommerce'),
        'description' => __('Check this box for custom option.', 'woocommerce'),
        'value'       => $checkbox_value,
        'cbvalue'     => 'yes',
        'checked'     => ( $checkbox_value === 'yes' ) ? 'checked' : ''
    ));
}
```

A Checkbox és a radio speciálisabb, mivel itt is kezelni kell a bejelölt értéket. Így a mentés a következő:

```
add_action( 'woocommerce_process_product_meta', 'save_custom_checkbox_field' );
function save_custom_checkbox_field( $post_id ) {
    $checkbox_value = isset( $_POST['_custom_product_checkbox'] ) ? 'yes' : 'no';
    update_post_meta( $post_id, '_custom_product_checkbox', $checkbox_value );
}
```

### Select (dropdown) típusú meta mező

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_select_field' );
function add_custom_select_field() {
    woocommerce_wp_select( array(
        'id'          => '_custom_product_select',
        'label'       => __('Custom Select', 'woocommerce'),
        'options'     => array(
            ''        => __('Please select', 'woocommerce'),
            'option1' => __('Option 1', 'woocommerce'),
            'option2' => __('Option 2', 'woocommerce')
        )
    ));
}
```
Mentés:

```
add_action( 'woocommerce_process_product_meta', 'save_custom_select_field' );
function save_custom_select_field( $post_id ) {
    // Ellenőrizzük, hogy a select mező értéke be van-e állítva
    if ( isset( $_POST['_custom_product_select'] ) ) {
        // Tisztítás
        $select_value = sanitize_text_field( $_POST['_custom_product_select'] );

        // Frissítsük a termék metaadatát a kiválasztott értékkel
        update_post_meta( $post_id, '_custom_product_select', $select_value );
    }
}
```

### Radio (Rádió gombok) típusú meta mező

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_radio_buttons' );
function add_custom_radio_buttons() {
    global $post;

    // Lekérjük a mentett értéket - lokális változó
    $saved_value = get_post_meta( $post->ID, '_custom_product_radio', true );

    // Rádiógombok megjelenítése
    echo '<div class="options_group">';
    echo '<p class="form-field _custom_product_radio_field">';
    echo '<label>' . __('Custom Radio Buttons', 'woocommerce') . '</label>';
    echo '<input type="radio" name="_custom_product_radio" value="option1" ' . checked( $saved_value, 'option1', false ) . '> ' . __('Option 1', 'woocommerce') . '<br>';
    echo '<input type="radio" name="_custom_product_radio" value="option2" ' . checked( $saved_value, 'option2', false ) . '> ' . __('Option 2', 'woocommerce');
    echo '</p>';
    echo '</div>';
}
```

Mentés:

A Radiogombok esetében van némi extra, mivel vissza kell töltenünk a bejelölt értéket, ezért valamivel eltérőbb lesz a többinél az alapértelmezett kódja
```
function save_custom_radio_buttons( $post_id ) {
    if ( isset( $_POST['_custom_product_radio'] ) ) {
        $radio_value = sanitize_text_field( $_POST['_custom_product_radio'] );
        update_post_meta( $post_id, '_custom_product_radio', $radio_value );
    } else {
    }
}
add_action( 'woocommerce_process_product_meta', 'save_custom_radio_buttons' );
```


### Speciális Multiple select mező

Amennyiben olyan mezőt szeretnél készíteni, amiben egy vagy több értéket is ki lehet jelölni, úgy felhasználhatjuk a WooCommerce által is használt select2 kiegészítőt
Itt két érték van "value1 stb stb", és mellé egy "Label" A label paraméterek lesznek azok amik a mezőben láthatóak lesznek a felhasználó számára, de az adatbázisban a value1 stb értékek kerülnek mentésre.

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_multiple_select_field' );
function add_custom_multiple_select_field() {
    global $post;

    // Lekérjük a mentett értékeket
    $saved_values = get_post_meta( $post->ID, '_custom_product_multiple_select', true );
    $saved_values = !empty($saved_values) ? explode(',', $saved_values) : array();

    // Opciók definiálása
    $options = array(
        'value1' => __('Label 1', 'woocommerce'),
        'value2' => __('Label 2', 'woocommerce'),
        'value3' => __('Label 3', 'woocommerce'),
        'value4' => __('Label 4', 'woocommerce')
    );

    echo '<p class="form-field">';
    echo '<label for="_custom_product_multiple_select">' . __('Custom Multiple Select', 'woocommerce') . '</label>';
    echo '<select id="_custom_product_multiple_select" name="_custom_product_multiple_select[]" class="wc-enhanced-select" multiple="multiple">';
    foreach ($options as $value => $label) {
        echo '<option value="' . esc_attr($value) . '" ' . (in_array($value, $saved_values) ? 'selected' : '') . '>' . esc_html($label) . '</option>';
    }
    echo '</select>';
    echo '</p>';
}
```

Mentés esetében, hogy ezek az értékek szűrő kompatibilisek legyenek az *implode* segítségével vesszőalapú elválasztási technikát alkalmazhatunk, így például ezeket az értékeket a JetSmartFilter képes lesz érézkelni, és szűrni:

```
add_action( 'woocommerce_process_product_meta', 'save_custom_multiple_select_field' );
function save_custom_multiple_select_field( $post_id ) {
    if ( isset( $_POST['_custom_product_multiple_select'] ) ) {
        $select_values = array_map('sanitize_text_field', $_POST['_custom_product_multiple_select']);
        $select_values = implode(',', $select_values); // Vesszővel elválasztott stringgé alakítjuk
        update_post_meta( $post_id, '_custom_product_multiple_select', $select_values );
    } else {
        delete_post_meta( $post_id, '_custom_product_multiple_select' );
    }
}
```
### Frontend megjelenítései metódusok

Fontos megjegyezni, hogy a frontend -en való kiíráskor alapértelmezetten a value azaz az adatbázisban tárolt értéket fogja megjeleníteni. Nézzünk meg egy shortcode példát az alapvető value értékek megjelenítésére, és egy label alapú megjelenítésre is. Megfelelő adatbázis kódolás esetében használhatsz a tárolt értékek esetében ékezetes értékeket is.

value alapú alapértelmezett megjelenítése:
**Shortcode: [custom_multiple_select]**

```
function display_custom_multiple_select_values() {
    global $post;

    // Ellenőrizzük, hogy van-e érvényes termék ID
    if ( !is_a( $post, 'WP_Post' ) || get_post_type( $post->ID ) !== 'product' ) {
        return '';
    }

    // Lekérjük a mentett értékeket
    $saved_values = get_post_meta( $post->ID, '_custom_product_multiple_select', true );
    if ( !empty($saved_values) ) {
        // Vesszővel elválasztva jelenítjük meg az értékeket
        return esc_html( implode( ', ', explode(',', $saved_values) ) );
    }

    return __('Nincs beállított érték', 'woocommerce');
}
add_shortcode( 'custom_multiple_select', 'display_custom_multiple_select_values' );
```

Label alapú megjelenítése:
**Shortcode: [custom_multiple_select_labels]**

```
function display_custom_multiple_select_labels() {
    global $post;

    // Ellenőrizzük, hogy van-e érvényes termék ID
    if ( !is_a( $post, 'WP_Post' ) || get_post_type( $post->ID ) !== 'product' ) {
        return '';
    }

    // Lekérjük a mentett értékeket
    $saved_values = get_post_meta( $post->ID, '_custom_product_multiple_select', true );
    if ( !empty($saved_values) ) {
        $saved_values = explode(',', $saved_values);

        // Opciók definiálása (ugyanazok, mint a mező létrehozásakor)
        $options = array(
            'value1' => __('Label 1', 'woocommerce'),
            'value2' => __('Label 2', 'woocommerce'),
            'value3' => __('Label 3', 'woocommerce'),
            'value4' => __('Label 4', 'woocommerce')
        );

        // Az értékek átkonvertálása label-ekre
        $labels = array();
        foreach ($saved_values as $value) {
            if (isset($options[$value])) {
                $labels[] = $options[$value];
            }
        }

        // Vesszővel elválasztva jelenítjük meg a label-eket
        return implode(', ', $labels);
    }

    return __('Nincs beállított érték', 'woocommerce');
}
add_shortcode( 'custom_multiple_select_labels', 'display_custom_multiple_select_labels' );
```

### Dátum alapú mező
Némi átalakításnak köszönhetően készíthetsz dátumválasztó meta mezőt is

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_date_field' );
function add_custom_date_field() {
    woocommerce_wp_text_input( array(
        'id'          => '_custom_product_date',
        'label'       => __('Custom Date', 'woocommerce'),
        'placeholder' => __('YYYY-MM-DD', 'woocommerce'),
        'type'        => 'date',
    ));
}
```

Egyszerű mentés:

```
add_action( 'woocommerce_process_product_meta', 'save_custom_date_field' );
function save_custom_date_field( $post_id ) {
    if ( isset( $_POST['_custom_product_date'] ) ) {
        $date_value = sanitize_text_field( $_POST['_custom_product_date'] );
        update_post_meta( $post_id, '_custom_product_date', $date_value );
    }
}
```

A Dátum mező esetében használhatunk timestamp / unix kódolásos mentést, ez hasznos lehet bizonyos esetekben, ilyenkor mind a mező mind a mentést átkell alakítanunk

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_date_field' );
function add_custom_date_field() {
    global $post;

    // Lekérjük a mentett időbélyeget és átalakítjuk dátummá
    $date_value = get_post_meta( $post->ID, '_custom_product_date', true );
    $date_value = !empty($date_value) ? date('Y-m-d', $date_value) : '';

    woocommerce_wp_text_input( array(
        'id'            => '_custom_product_date',
        'label'         => __('Custom Date', 'woocommerce'),
        'placeholder'   => __('YYYY-MM-DD', 'woocommerce'),
        'type'          => 'date',
        'value'         => $date_value, // Itt adjuk meg a konvertált dátumot
    ));
}
```

Időbélyeges mentés:

```
add_action( 'woocommerce_process_product_meta', 'save_custom_date_field' );
function save_custom_date_field( $post_id ) {
    if ( isset( $_POST['_custom_product_date'] ) ) {
        // Konvertáljuk a dátumot Unix időbélyeggé
        $date_value = strtotime( sanitize_text_field( $_POST['_custom_product_date'] ) );

        // Mentjük az időbélyeget a termék metaadataként
        update_post_meta( $post_id, '_custom_product_date', $date_value );
    }
}
```

## WooCommerce tab hookok

Az eddigi példában szereplő kódokat mind az általános tabfülbe helyeztük el. Lehetőség van bármelyik meglévőbe átrakni, sőt újakat is készíthetünk. A tabfül hookok a következők:

* woocommerce_product_options_general_product_data (általános)
* woocommerce_product_options_inventory_product_data (készlet)
* woocommerce_product_options_shipping (szállítás)
* woocommerce_product_options_related (kapcsolódó termékek)
* woocommerce_product_options_advanced (haladó)


## Egyedi mezők frontend megjelenítése fix hook poziciókkal

### Text input meta készítése, mentése, frontend megjelenítése

```
// egyszerű text input meta
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

// text input meta mentése (egyszerűsített verzió)

add_action( 'woocommerce_process_product_meta', 'save_custom_text_field' );
function save_custom_text_field( $post_id ) {
    if ( isset( $_POST['_custom_product_text_field'] ) ) {
        $custom_field_value = sanitize_text_field( $_POST['_custom_product_text_field'] );
        update_post_meta( $post_id, '_custom_product_text_field', $custom_field_value );
    }
}

// megjelenítés a termék single oldalon a rövid leírás alatt

add_action( 'woocommerce_single_product_summary', 'display_custom_text_field_on_product_page', 21 );
function display_custom_text_field_on_product_page() {
    global $product;
    $custom_field_value = get_post_meta( $product->get_id(), '_custom_product_text_field', true );

    if ( !empty($custom_field_value) ) {
        echo '<div class="custom-text-field">' . esc_html( $custom_field_value ) . '</div>';
    }
}
```

> [!WARNING]
> Egyes pagebuilderek, mint pédául az Elementor esetében elképzelhető hogy a hook-on kell változtatni, vagy mivel ilyenkor adott a termékoldalak pagebuilder-el való szerkesztése, készíthetünk shortcode-ot a megjelenítéshez.

A fenti kódban a megjelenítést a termék rövid leírás alá pozocionáltuk be a *add_action( 'woocommerce_single_product_summary', 'display_custom_text_field_on_product_page', 21 );* segítsgévéel. A WooCommerce számos további hook ot biztosít így például megjeleníthjük az egyedi mező értékét például a kosárhoz adás gomb alatt. **woocommerce_after_add_to_cart_form** segítségével a megjelenítendő adatunk a kosárhoz adás form alatt fog megjelenni. Csak cseréld ki a kódban szereplő **woocommerce_single_product_summary** erre: **woocommerce_after_add_to_cart_form**

További hookok-at vizuálisan megtalálod itt:
https://www.businessbloomer.com/woocommerce-visual-hook-guide-single-product-page/

További két példa a megjelenítésre:

### Label megjelenítése az érték előtt
Ebben a példában, nem csak az értéket jelenítjük meg a frontend-en, hanem egy labelt is elé helyezünk. Teljes kód:

```
// egyszerű text input meta
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

// text input meta mentése (egyszerűsített verzió)

add_action( 'woocommerce_process_product_meta', 'save_custom_text_field' );
function save_custom_text_field( $post_id ) {
    if ( isset( $_POST['_custom_product_text_field'] ) ) {
        $custom_field_value = sanitize_text_field( $_POST['_custom_product_text_field'] );
        update_post_meta( $post_id, '_custom_product_text_field', $custom_field_value );
    }
}

// megjelenítés

add_action( 'woocommerce_after_add_to_cart_form', 'display_custom_text_field_with_label', 21 );
function display_custom_text_field_with_label() {
    global $product;
    $custom_field_value = get_post_meta( $product->get_id(), '_custom_product_text_field', true );

    if ( !empty($custom_field_value) ) {
		echo '<div class="custom-meta-wrapper">'; // nyito div
        echo '<span class="custom-text-field-label">Mező értéke: </span>';
        echo '<span class="custom-text-field-value">' . esc_html( $custom_field_value ) . '</span>';
		echo '</div>'; // záró div
    }
}

```


