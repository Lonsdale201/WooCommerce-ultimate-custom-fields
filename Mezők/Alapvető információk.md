## WooCommerce egyedi mezők

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
