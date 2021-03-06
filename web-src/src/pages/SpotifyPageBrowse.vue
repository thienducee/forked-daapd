<template>
  <div>
    <tabs-music></tabs-music>

    <!-- New Releases -->
    <content-with-heading>
      <template slot="heading-left">
        <p class="title is-4">New Releases</p>
      </template>
      <template slot="content">
        <spotify-list-item-album v-for="album in new_releases" :key="album.id" :album="album">
          <template slot="actions">
            <a @click="open_album_dialog(album)">
              <span class="icon has-text-dark"><i class="mdi mdi-dots-vertical mdi-18px"></i></span>
            </a>
          </template>
        </spotify-list-item-album>
        <spotify-modal-dialog-album :show="show_album_details_modal" :album="selected_album" @close="show_album_details_modal = false" />
      </template>
      <template slot="footer">
        <nav class="level">
          <p class="level-item">
            <router-link to="/music/spotify/new-releases" class="button is-light is-small is-rounded">
              Show more
            </router-link>
          </p>
        </nav>
      </template>
    </content-with-heading>

    <!-- Featured Playlists -->
    <content-with-heading>
      <template slot="heading-left">
        <p class="title is-4">Featured Playlists</p>
      </template>
      <template slot="content">
        <spotify-list-item-playlist v-for="playlist in featured_playlists" :key="playlist.id" :playlist="playlist">
          <template slot="actions">
            <a @click="open_playlist_dialog(playlist)">
              <span class="icon has-text-dark"><i class="mdi mdi-dots-vertical mdi-18px"></i></span>
            </a>
          </template>
        </spotify-list-item-playlist>
        <spotify-modal-dialog-playlist :show="show_playlist_details_modal" :playlist="selected_playlist" @close="show_playlist_details_modal = false" />
      </template>
      <template slot="footer">
        <nav class="level">
          <p class="level-item">
            <router-link to="/music/spotify/featured-playlists" class="button is-light is-small is-rounded">
              Show more
            </router-link>
          </p>
        </nav>
      </template>
    </content-with-heading>
  </div>
</template>

<script>
import { LoadDataBeforeEnterMixin } from './mixin'
import ContentWithHeading from '@/templates/ContentWithHeading'
import TabsMusic from '@/components/TabsMusic'
import SpotifyListItemAlbum from '@/components/SpotifyListItemAlbum'
import SpotifyListItemPlaylist from '@/components/SpotifyListItemPlaylist'
import SpotifyModalDialogAlbum from '@/components/SpotifyModalDialogAlbum'
import SpotifyModalDialogPlaylist from '@/components/SpotifyModalDialogPlaylist'
import store from '@/store'
import * as types from '@/store/mutation_types'
import SpotifyWebApi from 'spotify-web-api-js'

const browseData = {
  load: function (to) {
    if (store.state.spotify_new_releases.length > 0 && store.state.spotify_featured_playlists.length > 0) {
      return Promise.resolve()
    }

    const spotifyApi = new SpotifyWebApi()
    spotifyApi.setAccessToken(store.state.spotify.webapi_token)
    return Promise.all([
      spotifyApi.getNewReleases({ country: store.state.spotify.webapi_country, limit: 50 }),
      spotifyApi.getFeaturedPlaylists({ country: store.state.spotify.webapi_country, limit: 50 })
    ])
  },

  set: function (vm, response) {
    if (response) {
      store.commit(types.SPOTIFY_NEW_RELEASES, response[0].albums.items)
      store.commit(types.SPOTIFY_FEATURED_PLAYLISTS, response[1].playlists.items)
    }
  }
}

export default {
  name: 'SpotifyPageBrowse',
  mixins: [ LoadDataBeforeEnterMixin(browseData) ],
  components: { ContentWithHeading, TabsMusic, SpotifyListItemAlbum, SpotifyListItemPlaylist, SpotifyModalDialogAlbum, SpotifyModalDialogPlaylist },

  data () {
    return {
      show_album_details_modal: false,
      selected_album: {},

      show_playlist_details_modal: false,
      selected_playlist: {}
    }
  },

  computed: {
    new_releases () {
      return this.$store.state.spotify_new_releases.slice(0, 3)
    },

    featured_playlists () {
      return this.$store.state.spotify_featured_playlists.slice(0, 3)
    }
  },

  methods: {
    open_album_dialog: function (album) {
      this.selected_album = album
      this.show_album_details_modal = true
    },

    open_playlist_dialog: function (playlist) {
      this.selected_playlist = playlist
      this.show_playlist_details_modal = true
    }
  }
}
</script>

<style>
</style>
