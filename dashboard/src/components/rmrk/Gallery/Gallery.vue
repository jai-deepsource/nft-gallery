<template>
  <div class="gallery">
    <!-- <b-field label="Owners">
      <b-select
        placeholder="Select an owner"
        :value="selectedOwner"
        @input="handleOwner"
      >
        <option v-for="option in allOwners" :value="option" :key="option">
          {{ option }}
        </option>
      </b-select>
    </b-field> -->
    <Search v-bind.sync="searchQuery" />
    <div>
      <div class="columns is-multiline">
        <div
          class="column is-4"
          v-for="nft in results"
          :key="nft.id"
        >
          <div class="card nft-card">
            <router-link :to="{ name: 'nftDetail', params: { id: nft.id }}" tag="div" class="nft-card__skeleton">
              <div class="card-image">
                <figure class="gallery__image-wrapper">
                  <img
                    :src="placeholder"
                    :data-src="nft.image"
                    :alt="nft.name"
                    class="lazyload gallery__image"
                    @error="onError"
                  />
                </figure>
              </div>

              <div class="card-content is-relative">
                <p
                  v-if="!isLoading"
                  class="title mb-0 is-4 has-text-centered"
                  :title="nft.name">
                  <router-link v-if="nft.count < 2" :to="{ name: 'nftDetail', params: { id: nft.id }}">
                    <div>
                      <div class="has-text-overflow-ellipsis middle">
                        {{ nft.name }}
                      </div>
                    </div>
                  </router-link>
                  <router-link v-else :to="{ name: 'collectionDetail', params: { id: nft.collection }}">

                    <div class="has-text-overflow-ellipsis">
                      {{ nft.name }}
                    </div>
                  </router-link>

                  <p
                    v-if="nft.count > 2"
                    :title="`${nft.count} items available in collection`"
                    class="is-absolute nft-collection-counter title is-6 is-color-pink"
                  >
                    「{{ nft.count }}」
                  </p>

                </p>
                <b-skeleton
                  :active="isLoading">
                </b-skeleton>
              </div>
            </router-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" >
import { Component, Prop, Vue, Watch } from 'vue-property-decorator';
import { getInstance } from '@/components/rmrk/service/RmrkService';
import { NFTWithMeta, NFT } from '../service/scheme';
import { defaultSortBy, sanitizeObjectArray } from '../utils';
import GalleryCardList from './GalleryCardList.vue'
import Search from './Search/SearchBar.vue'
import { basicFilter, basicAggQuery, expandedFilter } from './Search/query'
import axios from 'axios'
import Freezeframe from 'freezeframe'
import 'lazysizes'
import { SearchQuery } from './Search/types';

interface Image extends HTMLImageElement {
  ffInitialized: boolean
}

type NFTType = NFTWithMeta;
const components = { GalleryCardList, Search }

@Component({ components })
export default class Gallery extends Vue {
  private nfts: NFTType[] = [];
  private isLoading: boolean = true;
  private searchQuery: SearchQuery = {
    search: '',
    type: '',
    sortBy: { blockNumber: -1 }
  }
  private placeholder = require('@/assets/kodadot_logo_v1_transparent_400px.png')

  public async mounted() {
    const rmrkService = getInstance();

    this.setFreezeframe()

    if (!rmrkService) {
      return;
    }

    try {
      this.nfts = await rmrkService.getAllNFTs()
      .then(sanitizeObjectArray)
      .then(defaultSortBy);
      // this.collectionMeta();
    } catch (e) {
      console.warn(e);
    }

    this.isLoading = false;
  }

  get results() {
    // if (this.searchQuery) {
    //   return basicAggQuery(expandedFilter(this.searchQuery, this.nfts))
    // }

    return basicAggQuery(expandedFilter(this.searchQuery, this.nfts))
  }

  setFreezeframe() {
    document.addEventListener('lazybeforeunveil', async (e) => {
      const target = e.target as Image
      const src = target.dataset.src as string
      const image = await axios.head(src)
      const isGif = image.headers['content-type'] === 'image/gif'

      if (isGif && !target.ffInitialized) {
        const ff = new Freezeframe(target, {
          trigger: false,
          overlay: true,
          warnings: false
        })

        target.ffInitialized = true
      }
    })
  }

  onError(e: Event) {
    const target = e.target as Image
    target.src = this.placeholder
  }
}
</script>

<style lang="scss">
.gallery {
  &__image-wrapper {
    position: relative;
    margin: auto;
    padding-top: 100%;
    overflow: hidden;
    cursor: pointer;
  }

  &__image {
    bottom: 0;
    left: 0;
    position: absolute;
    right: 0;
    top: 50%;
    transition: all 0.3s;
    display: block;
    width: 100%;
    height: auto;
    transform: scale(1) translateY(-50%);

    &:hover {
      transform: scale(1.1) translateY(-50%);
    }
  }

  .ff-container {
    position: absolute;
    top: 0;
    height: 100%;
    overflow: hidden;

    .ff-overlay {
      z-index: 2;
    }

    .ff-image,
    .ff-canvas {
      top: 50%;
      height: auto;
      transform: translateY(-50%);
    }
  }

  .has-text-overflow-ellipsis {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }

  .is-float-right{
    float: right;
  }

  .is-color-pink {
    color: #d32e79;
  }

  .is-absolute {
    position: absolute;
  }

  .nft-collection-counter {
    top: 5px;
    right: -5px;
  }
}
</style>
