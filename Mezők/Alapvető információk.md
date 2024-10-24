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
* **placeholder:* Ezt nem kell kifejteni
* **desc_tip:** Tooltip engedélyezése
* **description:** Ez pedig a tooltip tartalma
