<?php
$post_id = get_the_ID();
$data    = aktie_get_offerte_data( $post_id );

$content_after = get_field( 'aktie_product_after_content' );

$show_form = false;
$staffel   = null;
$has_promo = false;

$staffel_items = array();

//checks if any item has a staffel_price set
foreach ( $data['items'] as $item ) {
	if ( $item['aktie_product_item_price_question'] ) {
		$show_form       = true;
		$staffel_items[] = $item;
	}
}

//checks if the amount isset
if ( $show_form ) {
	if ( $_POST ) {
		$show_form = false;
		$staffels  = aktie_get_offerte_range( $post_id, $_POST );
		$data      = aktie_get_offerte_data( $post_id, $_POST );
	}
}

$labels = get_field( 'aktie_staffel_form_labels' );

?>
<article id="post-<?php echo $post_id ?>" <?php post_class(); ?>>
    <div class="row">
        <div class="col-md-3">
			<?php get_template_part( 'template-parts/sidebars/sidebar', 'offerte' ); ?>
        </div>
        <div class="col-md-9 row">
            <header class="entry-header">
				<?php
				if ( is_singular() ) :
					if ( $content_after ):
						the_title( '<h1 class="entry-title col-md-12">', '</h1>' );
					else:
						the_title( '<h1 class="entry-title col-md-8">', '</h1>' );
					endif;
				else :
					if ( $content_after ):
						the_title( '<h2 class="entry-title col-md-12"><a href="' . esc_url( get_permalink() ) . '" rel="bookmark">', '</a></h2>' );
					else:
						the_title( '<h2 class="entry-title col-md-8"><a href="' . esc_url( get_permalink() ) . '" rel="bookmark">', '</a></h2>' );
					endif;
				endif;

				if ( 'post' === get_post_type() ) : ?>
                    <div class="entry-meta">
						<?php silverbee_starter_posted_on(); ?>
                    </div><!-- .entry-meta -->
					<?php
				endif; ?>
            </header><!-- .entry-header -->
            <div class="entry-content">
				<?php if ( $show_form ) : ?>
                    <div class="col-md-8">
						<?php echo wpautop( get_field( 'aktie_staffel_form_text' ) ); ?>
                        <form action="" method="post">
							<?php foreach ( $labels as $key => $label ) : ?>
                                <div class="form-group row">
                                    <label for="amount" class="col-md-6"
                                           style="text-align: right"><?php echo $label['aktie_staffel_form_label']; ?></label>
                                    <input type="number" class="col-md-6" id="amount-<?php echo $key ?>"
                                           name="amount-<?php echo $key ?>" required>
                                </div>
							<?php endforeach; ?>
                            <div class="row">
                                <button type="submit" class="btn button pull-right btn-primary">BEREKEN OFFERTE <span
                                            class="glyphicon glyphicon-arrow-right" aria-hidden="true"></span></button>
                            </div>
                        </form>
                    </div>
				<?php else: ?>
                <div class="col-md-8">
	                <?php
	                if ( ! empty( $data['items'] ) && $data['items'][0]['price'] != null ) :
		                $show_button = true;
		                $pdf = aktie_get_offerte_pdf( $post, $_POST );
		                $before = get_field( 'aktie_product_before_content', $post );
		                $_before = ( $before ) ? $before : "Hier is de offerte voor uw akte:";
		                ?>
                        <div class="offerte">
                            <h3 class="offerte-h3"><?php echo $_before ?></h3>
                            <div class="pull-right">
                                <a class="default" href="<?php echo $pdf['url'] ?>" title="PDF"><span
                                            class="glyphicon glyphicon-download" aria-hidden="true"></span></a>
                            </div>
                            <table>
                                <tr>
                                    <th><?php the_title(); ?></th>
                                </tr>
								<?php
								$staffel_counter = 0;

								foreach ( $data['items'] as $item ): ?>
									<?php if ( $item['aktie_product_promo_id'] ):
										$promo_id   = $item['aktie_product_promo_id'];
										$start_date = get_field( 'aktie_promo_start', $promo_id );
										$end_date   = get_field( 'aktie_promo_end', $promo_id );
										if ( $start_date <= date( 'Ymd' ) && date( 'Ymd' ) <= $end_date ) {
											$has_promo = true;
										} else {
											$has_promo = false;
										}
										?>
									<?php endif ?>
                                    <?php
                                        $product_id = str_replace( " ", "-", $item['aktie_product_item_name'] );
                                        $product_id = str_replace( "(", "", $product_id );
                                        $product_id = str_replace( ")", "", $product_id );
                                    ?>
                                    <tr id="<?php echo $product_id; ?>-tr">
                                        <td>
											<?php
											echo $item['aktie_product_item_name'];

											if ( $item['notes'] ) {
												echo '<sup>';
												foreach ( $item['notes'] as $note ) {
													echo '[' . $note . ']';
												}
												echo '</sup>';
											}
											?>
                                        </td>
                                        <td class="price"
                                            id="<?php echo $product_id; ?>-price">
                                            € <?php echo number_format_i18n( $item['price'], 2 ); ?>
                                        </td>
                                    </tr>
								<?php endforeach; ?>
                                <tr id="items-tr">
                                    <td>BTW (21%)</td>
                                    <td class="price" id="btw-td">
                                        € <?php echo number_format_i18n( $data['vat'], 2 ); // number_format_i18n( $data['vat'], 2 )
										?></td>
                                </tr>
                                <tr>
                                    <td><strong id="total_text_strong">Totaal bovenstaande</strong></td>
                                    <td class="price">
                                        <strong id="total_strong">€ <?php $total = (float) $data['total'] + (float) $data['vat'];
											echo number_format_i18n( $total, 2 ); ?></strong>
                                    </td>
                                </tr>
                            </table>
							<?php if ( $data['notes'] ) : ?>
                                <div class="notes">
                                    Overige details:
                                    <ul>
										<?php foreach ( $data['notes'] as $key => $note ) : ; ?>
                                            <li><?php echo '[' . $key . '] = ' . $note; ?></li>
										<?php endforeach; ?>
                                    </ul>
                                </div>
							<?php endif; ?>
							<?php if ( $has_promo == true ): ?>
                                <div class="input-group">
                                    <label for="offerte-discount">AKTIEcode: (optioneel)</label>

                                    <input type="text" class="form-control" id="offerte-discount"
                                           name="offerte-discount">
                                    <input type="hidden" value="<?php echo get_blog_details()->path ?>"
                                           id="vestiging"
                                           name="vestiging">
                                    <input type="hidden" value="<?php echo $post_id ?>" id="offerte_id"
                                           name="offerte_id">

                                    <span class="input-group-btn">
                                    <button style="margin-top:26px !important;" class="btn btn-default"
                                            id="checkactiecode" name="checkactiecode" onclick="check_actiecode()">
                                        Controleer AKTIEcode
                                    </button>
                                    </span>
                                </div>
							<?php endif ?>
                        </div>
					<?php endif; ?>
					<?php if ( $show_button ): ?>
                        <button class="btn button pull-right btn-primary" data-toggle="modal"
                                data-target="#offerte-form">ACCEPTEER OFFERTE <span
                                    class="glyphicon glyphicon-arrow-right" aria-hidden="true"></span></button>
					<?php endif ?>
                    <div class="content">
		                <?php the_content(); ?>
                    </div>
                </div>
                <div class="col-md-4">
                    <div class="after-content">
						<?php if ( $content_after ) : ?>
							<?php echo $content_after; ?>
						<?php else: ?>
                            <style>
                                @media (min-width: 992px) {
                                    .after-content {
                                        margin-top: -83px;
                                    }
                                }
                            </style>
							<?php get_template_part( 'template-parts/sidebars/sidebar', 'contact' ); ?>
						<?php endif; ?>
                    </div>
                </div>
            </div>
			<?php endif; ?>
        </div>
    </div><!-- .entry-content -->

    <footer class="entry-footer">
		<?php silverbee_starter_entry_footer(); ?>
    </footer><!-- .entry-footer -->
</article><!-- #post-<?php echo $post_id; ?> -->
<?php get_template_part( 'template-parts/modals/modal', 'offerte' ); ?>
<script>
    (function ($) {
        check_actiecode = function () {
            $.ajax({
                type: "GET",
                url: $('#vestiging').attr('value') + "wp-json/aktie/v1/offerte/" + $('#offerte_id').attr('value') +
                "/aktiecode/" + $('#offerte-discount').attr('value'),
                success: function (result) {
                    console.log(result);

                    var total_vat = parseFloat(result['vat'].replace(",", ".")) + parseFloat(result['total'].replace(",", "."));
                    total_vat = total_vat.toString().replace(".", ",");

                    $('#total_strong').html("€ " + total_vat);
                    if (result['discount']) {
                        var product = result['product_name'].replace(/[\(\)]/g, "");
                        product = product.replace(/\s/g, "-");

                        $("#" + product + "-price").html('<del>' + $("#" + product + "-price").html() + '</del>');
                        $("#" + product + "-tr").after("<tr><td>--- Promocode: " + $('#offerte-discount').attr('value') + "</td><td>€ " + result['discount'] + "</td></tr>");
                        $("#btw-td").html("€ " + result['vat']);
                        $("#checkactiecode").attr('disabled', true);
                        $(".modal-footer").after("<input type='hidden' id='offerte-discount' name='offerte-discount' value=" + $('#offerte-discount').attr('value') + ">");
                    } else {
                        window.alert("AKTIEcode werd niet herkend, probeer nogmaals.");
                    }
                },
                error: function (result) {
                    console.log(result);
                }
            });
        }
    })(jQuery);
</script>
