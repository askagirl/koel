<template>
    <section id="extra" :class="{ showing: state.showExtraPanel }">
        <div class="tabs">
            <div class="header clear">
                <a @click.prevent="currentView = 'lyrics'"
                    :class="{ active: currentView === 'lyrics' }">Lyrics</a>
                <a @click.prevent="currentView = 'artistInfo'"
                    :class="{ active: currentView === 'artistInfo' }">Artist</a>
                <a @click.prevent="currentView = 'albumInfo'"
                    :class="{ active: currentView === 'albumInfo' }">Album</a>
            </div>

            <div class="panes">
                <lyrics :song="song" v-ref:lyrics v-show="currentView === 'lyrics'"></lyrics>
                <artist-info :artist="song.artist" :mode="'sidebar'" v-ref:artist-info v-show="currentView === 'artistInfo'">
                </artist-info>
                <album-info :album="song.album" :mode="'sidebar'" v-ref:album-info v-show="currentView === 'albumInfo'">
                </album-info>
            </div>
        </div>
    </section>
</template>

<script>
    import isMobile from 'ismobilejs';
    import { invokeMap } from 'lodash';
    import $ from 'jquery';

    import lyrics from './lyrics.vue';
    import artistInfo from './artist-info.vue';
    import albumInfo from './album-info.vue';
    import preferences from '../../../stores/preference';
    import songStore from '../../../stores/song';
    import songInfoService from '../../../services/info/song';

    export default {
        components: { lyrics, artistInfo, albumInfo },

        data() {
            return {
                song: songStore.stub,
                state: preferences.state,
                currentView: 'lyrics',
            };
        },

        watch: {
            /**
             * Watch the "showExtraPanel" property to add/remove the corresponding class
             * to/from the html tag.
             * Some element's CSS can then be controlled based on this class.
             */
            'state.showExtraPanel': function (newVal) {
                if (newVal && !isMobile.any) {
                    $('html').addClass('with-extra-panel');
                } else {
                    $('html').removeClass('with-extra-panel');
                }
            },
        },

        ready() {
            // On ready, add 'with-extra-panel' class.
            if (!isMobile.any) {
                $('html').addClass('with-extra-panel');
            }

            if (isMobile.phone) {
                // On a mobile device, we always hide the panel initially regardless of
                // the saved preference.
                preferences.showExtraPanel = false;
            }
        },

        methods: {
            /**
             * Reset all self and applicable child components' states.
             */
            resetState() {
                this.currentView = 'lyrics';
                this.song = songStore.stub;
                invokeMap(this.$refs, 'resetState');
            },
        },

        events: {
            'main-content-view:load': function (view) {
                // Hide the panel away if a main view is triggered on mobile.
                if (isMobile.phone) {
                    preferences.showExtraPanel = false;
                }

                return true;
            },

            'song:played': function (song) {
                songInfoService.fetch(this.song = song);
            },

            'koel:teardown': function () {
                this.resetState();
            },
        },
    };
</script>

<style lang="sass">
    @import "../../../../sass/partials/_vars.scss";
    @import "../../../../sass/partials/_mixins.scss";

    #extra {
        flex: 0 0 $extraPanelWidth;
        padding: 24px 16px $footerHeight;
        background: $colorExtraBgr;
        max-height: calc(100vh - #{$headerHeight + $footerHeight});
        display: none;
        color: $color2ndText;
        overflow: auto;
        -ms-overflow-style: -ms-autohiding-scrollbar;

        html.touchevents & {
            // Enable scroll with momentum on touch devices
            overflow-y: scroll;
            -webkit-overflow-scrolling: touch;
        }

        &.showing {
            display: block;
        }

        h1 {
            font-weight: $fontWeight_UltraThin;
            font-size: 2.2rem;
            margin-bottom: 16px;
            line-height: 2.8rem;
        }

        @media only screen and (max-width : 1024px) {
            position: fixed;
            height: calc(100vh - #{$headerHeight + $footerHeight});
            padding-bottom: $footerHeight; // make sure the footer can never overlap the content
            width: $extraPanelWidth;
            z-index: 5;
            top: $headerHeight;
            right: -100%;
            transition: right .3s ease-in;

            &.showing {
                right: 0;
            }
        }

        @media only screen and (max-width : 667px) {
            width: 100%;
        }
    }
</style>
