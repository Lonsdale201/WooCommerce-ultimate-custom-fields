## WooCommerce egyedi mezők - általános leírás

A WooCommerce termékekhez létrehozhatunk saját mezőket. Ezt megtehetjük számos bővítménnyel (Advanced Custom Fields, JetEngine stb), vagy kód segítségével is. Az egyedi mezőket felhasználhatjuk, hogy további extra adatokat tudjunk társítani az adott termékekhez. A WooCommerce API segítségével 
könnyedén létrehozhatunk további mezőket, sőt meghatározhatjuk melyik terméktabfülhöz rendelje hozzá. Az APi segítségével létrehozhatunk új füleket is, új terméktípust is, és az egyedi mezőniket kondícionálisan akár ezekbe is besorolhatjuk. 

### Alapvető mezőtípusok

* Text
* Textarea
* Number
* Url
* Date
* Checkbox
* Radio
* Select (dropdown)

### Mező létrehozási minta

```

add_action('woocommerce_product_options_general_product_data', 'custom_product_text_field');
function custom_product_text_field() {
    woocommerce_wp_text_input(
        array(
            'id' => '_custom_text_field', // Egyedi mező azonosítója
            'label' => __('Egyedi szövegmező', 'woocommerce'), // Mező felirata
            'placeholder' => __('Add meg a leírást', 'woocommerce'), // Placeholder szöveg
            'desc_tip' => 'true', // Tooltip szöveg megjelenítése
            'description' => __('Adjon meg egy egyedi értéket a termékhez.', 'woocommerce'), // Tooltip szöveg tartalma
        )
    );
}

```

Hook használata: A **woocommerce_product_options_general_product_data** hookkal illesztjük be a mezőt a WooCommerce admin felületére, a termékek szerkesztési oldalára.


**woocommerce_wp_text_input():** Ez egy beépített WooCommerce funkció, amely egy szövegmezőt ad hozzá az admin felülethez. Alatta pedig a különböző paraméterek:
* **id:** Az egyedi mező azonosítója. Ezt az azonosítót használjuk a mező mentéséhez és megjelenítéséhez.
* **label:** A mező címkéje, amit az admin felületén látunk.
* **placeholder:** Ezt nem kell kifejteni
* **desc_tip:** Tooltip engedélyezése
* **description:** Ez pedig a tooltip tartalma

Az egyes tabfülekbe való elhelyezésnél az alábbi Hook-okat tudod használni:

* woocommerce_product_options_general_product_data (általános tab)
* woocommerce_product_options_dimensions (Szállítás tab)
* woocommerce_product_options_inventory_product_data (készlet)
* woocommerce_product_options_advanced (haladó)

### Mező mentése

A következő fontos lépés hogy biztosítsuk a mező mentését, és annak kezelését. Ehhez szintén egy WC áltla biztosított Hook ot tudunk használni ***woocommerce_process_product_meta*** a ***sanitize_text_field*** segítségével tisztítjuk a mezőt mentés előtt.
Ha a mező nem tartalmaz értéket, tehát üres marad, akkor töröljük a meta adatot. Az ***update_post_meta*** gyári WordPress funkció ami a meta mentésére szolgál. Az első érték a termék azonosítója (post id), a második az áltlaunk látrehozott egyedi mező azonosítója, a harmadik pedig annak értéke.

```

// Egyedi mező értékének mentése
add_action('woocommerce_process_product_meta', 'save_custom_product_text_field');

function save_custom_product_text_field($post_id) {
    // Ellenőrizzük, hogy van-e adat a mezőben
    $custom_field_value = isset($_POST['_custom_text_field']) ? sanitize_text_field($_POST['_custom_text_field']) : '';

    // Meta adat elmentése a termékhez
    if (!empty($custom_field_value)) {
        update_post_meta($post_id, '_custom_text_field', esc_attr($custom_field_value));
    } else {
        delete_post_meta($post_id, '_custom_text_field'); // Ha üres, töröljük az adatot
    }
}
```

### Mező megjelenítése
Gyakran az egyedi mezőnk extra termék információként szolgál, ezért annak adatát megakarjuk jeleníteni a termék adatlapján. Két gyakori formája is van. Az egyik egy fix hook segítségével, a másik shortcode használata. A Shortcode akkor hasznos, ha olyan eszközt hasznákunk, mint például Elementor, vagy Bricks, aminek a segítségével saját sablont tudunk létrehozni a termékeknek, és így könnyedén elhelyezhetjük a megjelenítendő adatot, bárhol a termék adatlapon belül.

- Példa 1 gyári hook használatával a mező megjelenítése a frontend-en.

A legfontosabb, hogy kiválasszuk a megfelelő elhelyezésre/megjelenítésre szolgáló hook-ot.
Vizuális hook segédlet: https://www.businessbloomer.com/woocommerce-visual-hook-guide-single-product-page/
A kódban a ***woocommerce_before_add_to_cart_button*** használjuk, amely a meta tartalmát a kosárhoz adás gomb előtt fogja megjeleníteni.

```
add_action('woocommerce_before_add_to_cart_button', 'display_custom_product_text_field');

function display_custom_product_text_field() {
    global $post;

    // Az egyedi mező értékének lekérése
    $custom_field_value = get_post_meta($post->ID, '_custom_text_field', true);

    if (!empty($custom_field_value)) {
        echo '<p class="custom-field-text">' . esc_html($custom_field_value) . '</p>';
    }
}
```
