## Egyedi tabfül regisztrálás

A következő példában regisztrálunk egy új egyedi tabfület, és egy text metát fogunk benne elhelyezni.

```
add_filter( 'woocommerce_product_data_tabs', 'add_custom_product_data_tab' );
function add_custom_product_data_tab( $tabs ) {
    $tabs['custom_tab'] = array(
        'label'    => __('Kiegészítők', 'woocommerce'),
        'target'   => 'custom_product_data',
        'class'    => array(),
        'priority' => 80,
    );
    return $tabs;
}

add_action( 'woocommerce_product_data_panels', 'add_custom_product_data_fields' );
function add_custom_product_data_fields() {
    echo '<div id="custom_product_data" class="panel woocommerce_options_panel">';
    woocommerce_wp_text_input( array(
        'id'          => '_custom_product_text',
        'label'       => __('Egyedi szövegmező', 'woocommerce'),
        'description' => __('Ez egy egyedi szövegmező a Kiegészítők fülön.', 'woocommerce'),
        'desc_tip'    => true,
    ));
    echo '</div>';
}


add_action( 'woocommerce_process_product_meta', 'save_custom_product_text_field' );
function save_custom_product_text_field( $post_id ) {
    $custom_text = isset( $_POST['_custom_product_text'] ) ? sanitize_text_field( $_POST['_custom_product_text'] ) : '';
    if ( ! empty( $custom_text ) ) {
        update_post_meta( $post_id, '_custom_product_text', $custom_text );
    } else {
        delete_post_meta( $post_id, '_custom_product_text' );
    }
}

```

A kód három szakaszból áll. Az elsőben regisztrálunk egy új fület *Kiegészítők* néven. A második bekezdésben egy text metát helyezük el benne, a harmadik szakaszban pedig gondoskodunk ennek mentéséről. Most az egyedi tabfül lesz az "újdonság". Nézzük meg annak paramétereit mit jelentenek


* LABEL    => Ez lesz a tabfülnek neve, amit a felhasználó látni fog a termékszerkesztőben
* TARGET   => Ez az attirbutum azért szükséges, mert amikor az új létrehozott tabfülünkre rákattintanak a woocommerce megkeresi a **#custom_product_data** azonosítójú elemet és a tartalmat oda tölti be. Tehát nem a tab elhelyezésére használják, hanem annak tartalmának helyét jelöli.
* CLASS    => Bár üresen hagytuk, azonban egyedi osztály is megadhatunk a tab elemnek ha szükséges. Sőt megfelelő dashicon megadásával módosíthatjuk a tabfül ikonját is. Ehhe viszont kell egy kevés CSS is.
* PRIORITY => Piroritás. Minél kisebb a szám úgy fog előrébb kerülni a tabfülek sorrnejdében az új létrehozott fül.

### További lehetőségek

Jelenleg az új tabfül minden típusú termék esetében elérhető, egyszerű, variálható stb. azonban beállítható az, hogy csak specifikus esetékben legyen elérhető, például csak a variálható termékek esetében

```
add_filter( 'woocommerce_product_data_tabs', 'add_custom_product_data_tab' );
function add_custom_product_data_tab( $tabs ) {
    $tabs['custom_tab'] = array(
        'label'    => __('Kiegészítők', 'woocommerce'),
        'target'   => 'custom_product_data',
        'class'    => array('show_if_variable'), 
        'priority' => 80,
    );
    return $tabs;
}
```
A változtatás itt, a class attributumban történt, a show_if_variable egy beépített opció a WooCommerce részéről amit felhasználhatunk arra, hogy meghatározzuk ezáltal hogy a tabfül csak a variálható termékek esetében jelenjen meg.
További metódusok ide vonatkozóan:

* show_if_variable csak variálható termékek esetében
* show_if_simple csak egyszerű termékek esetében
* show_if_grouped csak csoportosított termékek esetében
* show_if_external csak külső/partner termékek esetében
* show_if_downloadable csak letölthető termékek esetében
* show_if_virtual csak virtuális termékek esetében

Ha egyszerre többet szeretnél megadni (vagy reláció): 'class'    => array('show_if_variable', 'show_if_downloadable'),
