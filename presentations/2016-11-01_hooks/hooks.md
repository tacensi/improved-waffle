Hooks, actions & filters: Extendendo WP do jeito certo
======================================================

## Hooks

* Tradução: Gancho
* Hook: a piece of metal or other material, curved or bent back at an angle, for catching hold of or hanging things on.
* To hook: attach or fasten with a hook or hooks.

## Hooks no WP

* Serve para pendurar, prender ações individuais para alterar ou acrescentar conteúdo ou funcionalidades ao WP
* Dois tipos:
	* Actions: Adiciona funcionalidade
	* Filters: Modifica algum tipo de conteúdo
	* Tl;dr: Actions fazem alguma coisa. Filters modificam alguma coisa.

## Actions

### Exemplos no _core_:

#### Actions: função `wp_head();`:

```
function wp_head() {
    /**
     * Prints scripts or data in the head tag on the front end.
     *
     * @since 1.5.0
     */
    do_action( 'wp_head' );
}
```

#### Filters: função `the_content();`:

```
function the_content( $more_link_text = null, $strip_teaser = false) {
    $content = get_the_content( $more_link_text, $strip_teaser );

    /**
     * Filters the post content.
     *
     * @since 0.71
     *
     * @param string $content Content of the current post.
     */
    $content = apply_filters( 'the_content', $content );
    $content = str_replace( ']]>', ']]&gt;', $content );
    echo $content;
}
```

## Por quê!?

* Mude o comportamento do WP sem mexer em arquivos do core;
* Não é possível ter temas netos. Altere temas filho sem alterar os arquivos do tema. Ex: temas que usam frameworks;
* Separe os hooks em plugins para modulação e fácil ativação/desativação;
* Mantenha tema/WP/plugins funcionando com updates;

## Adicionando um hook

* `add_action( 'tag', 'function', 'priority', 'parameters' )`
* `add_filter( 'tag', 'function', 'priority', 'parameters' )`

### Tag

O nome do hook, onde a função de callback será chamada.

#### Filter
```
function the_content( $more_link_text = null, $strip_teaser = false) {
    $content = get_the_content( $more_link_text, $strip_teaser );

    /**
     * Filters the post content.
     *
     * @since 0.71
     *
     * @param string $content Content of the current post.
     */
    $content = apply_filters( 'the_content', $content );
    $content = str_replace( ']]>', ']]&gt;', $content );
    echo $content;
}
```

```
add_filter( 'the_content', function( $content ) {
	return str_replace( 'Valdemort', 'Aquele-Que-Não-Deve-Ser-Nomeado', $content );
}, 30 );
```

#### Action

```
function wp_head() {
    /**
     * Prints scripts or data in the head tag on the front end.
     *
     * @since 1.5.0
     */
    do_action( 'wp_head' );
}
```

```
add_action( 'wp_head', function(){
	echo "OpenGraph Tags...";
}, 10 );
```

### Função de callback

Nome da sua função. Sem mistério aqui.

#### Action

__`tacensi_add_og_meta`__

```
add_action( 'wp_head', 'tacensi_add_og_meta', 10 );
function tacensi_add_og_meta(){
	echo "OpenGraph Tags...";
}
```

#### Filter

__`tacensi_never_say_his_name`__

```
add_filter( 'the_content', 'tacensi_never_say_his_name', 30 );
function tacensi_never_say_his_name( $content ) {
	return str_replace( 'Voldemort', 'Aquele-Que-Não-Deve-Ser-Nomeado', $content );
}
```

### Prioridade

Ordem de execução de todas as funções penduradas naquele hook. São executadas em ordem crescente (10, 20, 30, etc...).

#### Action

`wp_includes/default_filters.php:205`:

```
// Actions
add_action( 'wp_head', '_wp_render_title_tag',            1     );
add_action( 'wp_head', 'wp_enqueue_scripts',              1     );
add_action( 'wp_head', 'feed_links',                      2     );
add_action( 'wp_head', 'feed_links_extra',                3     );
add_action( 'wp_head', 'rsd_link'                               );
add_action( 'wp_head', 'wlwmanifest_link'                       );
add_action( 'wp_head', 'adjacent_posts_rel_link_wp_head', 10, 0 );
add_action( 'wp_head', 'locale_stylesheet'                      );
add_action( 'publish_future_post', 'check_and_publish_future_post',   10, 1 );
add_action( 'wp_head',             'noindex',                          1    );
add_action( 'wp_head',             'print_emoji_detection_script',     7    );
add_action( 'wp_head',             'wp_print_styles',                  8    );
add_action( 'wp_head',             'wp_print_head_scripts',            9    );
add_action( 'wp_head',             'wp_generator'                           );
add_action( 'wp_head',             'rel_canonical'                          );
add_action( 'wp_footer',           'wp_print_footer_scripts',         20    );
add_action( 'wp_head',             'wp_shortlink_wp_head',            10, 0 );
add_action( 'template_redirect',   'wp_shortlink_header',             11, 0 );
add_action( 'wp_print_footer_scripts', '_wp_footer_scripts'                 );
add_action( 'init',                'check_theme_switched',            99    );
add_action( 'after_switch_theme',  '_wp_sidebars_changed'                   );
add_action( 'wp_print_styles',     'print_emoji_styles'                     );
```

#### Filter

Deve-se tomar mais cuidado com a ordem nos filtros, já que a modificação do conteúdo pode não sair do jeito desejado.

`wp_includes/default_filters.php:145`:

```
add_filter( 'comment_text', 'wptexturize'            );
add_filter( 'comment_text', 'convert_chars'          );
add_filter( 'comment_text', 'make_clickable',      9 );
add_filter( 'comment_text', 'force_balance_tags', 25 );
add_filter( 'comment_text', 'convert_smilies',    20 );
add_filter( 'comment_text', 'wpautop',            30 );
```

### Número de parâmetros

Quantos parâmetros a função aceita. Se não há uma documentação devemos cavar no fonte.

#### Action

`wp_includes/default_filters.php:288`:

`add_action( 'post_updated',      'wp_check_for_changed_slugs', 12, 3 );`

`wp-includes/post.php:5286`

```
/**
 * @param int     $post_id     Post ID.
 * @param WP_Post $post        The Post Object
 * @param WP_Post $post_before The Previous Post Object
 * @return int Same as $post_id
 */
function wp_check_for_changed_slugs( $post_id, $post, $post_before ) {
	(...)
}
```


#### Filtro

`wp_includes/default_filters.php:186`:

`add_filter( 'sanitize_title', 'sanitize_title_with_dashes', 10, 3 );`

```
/**
 * @param string $title The title to be sanitized.
 * @param string $raw_title Optional. Not used.
 * @param string $context Optional. The operation for which the string is sanitized.
 * @return string The sanitized title.
 */
function sanitize_title_with_dashes( $title, $raw_title = '', $context = 'display' ) {
	(...)
```

## Removendo Actions e Filters

### Actions

Usar função `remove_action()`. Ex:

```
// remove unncessary header info
function tacensi_remove_header_info() {
	remove_action( 'wp_head', 'rsd_link' );
	remove_action( 'wp_head', 'wlwmanifest_link' );
	remove_action( 'wp_head', 'wp_generator' );
	remove_action( 'wp_head', 'start_post_rel_link' );
	remove_action( 'wp_head', 'index_rel_link' );
	remove_action( 'wp_head', 'adjacent_posts_rel_link' );
}
add_action( 'init', 'tacensi_remove_header_info' );
```

## nvr

Exemplos da Vida Real:

### Limpando `<head>`:


```
// remove unncessary header info
function tacensi_remove_header_info() {
	remove_action( 'wp_head', 'rsd_link' );
	remove_action( 'wp_head', 'wlwmanifest_link' );
	remove_action( 'wp_head', 'wp_generator' );
	remove_action( 'wp_head', 'start_post_rel_link' );
	remove_action( 'wp_head', 'index_rel_link' );
	remove_action( 'wp_head', 'adjacent_posts_rel_link' );
}
add_action( 'init', 'tacensi_remove_header_info' );
```

### WooCommerce

Modificando o WooCommerce:

#### Action

```
add_action( 'woocommerce_before_shop_loop', 'tacensi_add_promo' );
function tacensi_add_promo() {
	?>
	<div class="banner-promo">
		<h2>O patrão está maluco!</h2>
		<p>Somente hoje, tudo pela metade do dobro!</p>
	</div>
	<?php
}
```

#### Filter

```
    add_filter( 'woocommerce_free_price_html', 'tacensi_free_price' );
function tacensi_free_price( $price ) {
	$price = '<span class="amount">' .
		__( 'Totalmente de GRÁTIS!', 'tacensi' ) .
		'</span>';
	return $price;
}
```

### Twentyseventeen

Filtro com número de seções na front page:

```
add_filter( 'twentyseventeen_front_page_sections', function(){
    return 5;
});
```

http://adambrown.info/p/wp_hooks
