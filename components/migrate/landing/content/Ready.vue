<template>
  <div class="mt-8 pt-4">
    <div class="flex relative section-title">
      <img src="/migrate/state-ready.svg" alt="Ready" />
      <p>{{ $t('migrate.ready.title') }}</p>
    </div>

    <div class="text-k-grey mt-2">
      {{ $t('migrate.ready.desc') }}
    </div>

    <div class="collection">
      <div
        v-for="collection in collections"
        :key="collection.id"
        class="collection-card"
        :class="{
          hidden: !entities[collection.id]?.id,
        }">
        <div
          class="collection-card-banner"
          :style="{
            backgroundImage: `url(${entities[collection.id]?.image})`,
          }"></div>
        <div
          class="collection-card-avatar"
          :style="{
            backgroundImage: `url(${entities[collection.id]?.image})`,
          }"></div>

        <div class="collection-card-info">
          <div class="flex justify-between items-center">
            <div>
              <p class="is-size-5 font-bold">
                {{ collection.name }}
              </p>
              <p>
                <span class="text-k-grey mr-2">
                  {{ $t('migrate.ready.status') }}
                </span>
                <span>
                  {{ collection.nftsOwned?.length }}/{{
                    collection.nfts?.length
                  }}
                  Items
                </span>
              </p>
            </div>
            <div>
              <NeoButton
                variant="pill"
                @click="
                  toReview({
                    collectionId: collection.id,
                    itemCount: collection.nftsOwned?.length,
                  })
                ">
                {{ $t('migrate.ready.cta') }}
              </NeoButton>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { NeoButton } from '@kodadot1/brick'
import { toReview, useReadyItems } from '@/composables/useMigrate'

const { collections, entities } = useReadyItems()
</script>
