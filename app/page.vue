<template>

    <div>
        <div class="uk-alert uk-alert-danger" v-if="error">{{ error }}</div>
        <div ref="container" v-html="page"></div>
    </div>

</template>

<script>

    import $ from 'jquery';

    export default {

        data: () => ({
            loading: false,
            page: null,
            error: null,
            cache: {},
            extension: 'html'
        }),

        watch: {

            $route: {

                handler() {

                    var page = this.$route.params.page;

                    scrollTo(0, 0);

                    this.loading = true;
                    this.error = null;

                    this.$parent.page = page;

                    var defer = $.Deferred();

                    defer
                        .then(page => this.setPage(page), () => this.error = 'Failed loading page')
                        .always(() => this.loading = false);

                    if (this.cache[page]) {
                        defer.resolve(this.cache[page]);
                        return;
                    }

                    $.get(`pages/${page}.${this.extension}`, {nc: Math.random()}).then(content => {

                        this.cache[page] = content;
                        defer.resolve(content);

                    }, err => reject(err));

                },

                immediate: true
            },

            page() {

                this.$nextTick(() => {

                    if (location.hash && $(location.hash).length) {
                        scrollTo(0, $(location.hash).offset().top);
                    }

                });

            }

        },

        methods: {

            setPage(page) {
                this.page = page;
                this.$nextTick(() => $('<div>').append(page).find('script').appendTo(this.$refs.container));
            }

        }

    }

</script>
