<template>
  <div class="fixed-size border">
    <div
      class="fixed-top-left border px-4 py-2 bg-background-color z-[1] whitespace-nowrap">
      {{ $t('mint.unlockable.yourVariation') }}
    </div>

    <BaseMediaItem
      :src="sanitizeIpfsUrl(displayUrl)"
      :mime-type="generativeImageUrl ? 'text/html' : ''"
      preview
      is-detail
      class="border-b" />
    <div class="flex justify-center items-center py-6">
      <NeoButton
        v-if="isLoading"
        class="border-k-grey hover-button fixed-width pointer-events-auto cursor-wait"
        rounded
        no-shadow
        disabled>
        {{ $t('mint.unlockable.generating') }}
        <NeoIcon icon="circle-notch" spin />
      </NeoButton>
      <NeoButton
        v-else
        class="border-k-grey hover-button fixed-width"
        rounded
        no-shadow
        icon="arrow-rotate-left"
        @click="generateNft()">
        {{ $t('mint.unlockable.variations') }}
      </NeoButton>

      <a
        v-safe-href="sanitizeIpfsUrl(displayUrl)"
        class="flex items-center text-k-blue hover:text-k-blue-hover fixed-right"
        rel="nofollow noopener noreferrer"
        target="_blank"
        role="link">
        {{ $t('img') }}
        <NeoIcon icon="arrow-up-right" class="ml-1" />
      </a>
    </div>
  </div>
</template>

<script setup lang="ts">
import { NeoButton, NeoIcon } from '@kodadot1/brick'
import { sanitizeIpfsUrl } from '@/utils/ipfs'
import { getRandomIntFromRange } from '../unlockable/utils'
import { blake2AsHex, encodeAddress } from '@polkadot/util-crypto'

const props = defineProps<{
  minted: number
  content: string
  image?: string
}>()

const emit = defineEmits(['select'])

const { accountId } = useAuth()

const STEP = 64
const entropyRange = computed<[number, number]>(() => [
  STEP * props.minted,
  STEP * (props.minted + 1),
])

const getHash = (isDefault?: boolean) => {
  const ss58Format = isDefault
    ? entropyRange.value[0]
    : getRandomIntFromRange(entropyRange.value[0], entropyRange.value[1])

  // https://github.com/paritytech/ss58-registry/blob/30889d6c9d332953a6e3333b30513eef89003f64/ss58-registry.json#L1292C17-L1292C22
  const initialValue = accountId.value
    ? encodeAddress(accountId.value, ss58Format)
    : String(Date.now() << ss58Format)
  return blake2AsHex(initialValue, 256, null, true)
}

const generativeImageUrl = ref(
  accountId.value ? `${props.content}/?hash=${getHash(true)}` : '',
)

const isLoading = ref(false)

const displayUrl = computed(() => {
  return generativeImageUrl.value || props.image
})
const generateNft = (isDefault: boolean = false) => {
  isLoading.value = true
  const metadata = `${props.content}/?hash=${getHash(isDefault)}`
  generativeImageUrl.value = metadata
  emit('select', generativeImageUrl.value)

  setTimeout(() => {
    isLoading.value = false
  }, 3000)
}

watch(
  accountId,
  () => {
    generateNft(true)
  },
  {
    immediate: true,
  },
)
</script>

<style scoped lang="scss">
@import '@/assets/styles/abstracts/variables';

.fixed-size {
  width: 36rem;
  height: min-content;
  position: relative;

  @include mobile {
    width: 100% !important;
    height: 100% !important;
    max-width: 35rem;
  }
  @include tablet-only {
    width: 48rem;
  }
}

.fixed-top-left {
  @apply absolute rounded-[3rem] left-[26px] -top-3.5;

  @include mobile {
    left: 50%;
    transform: translateX(-50%);
  }
}

.hover-button:hover {
  background: unset;
  @include ktheme() {
    border-color: theme('border-color') !important;
  }
}

.fixed-right {
  position: absolute;
  right: 2rem;
  @include mobile {
    right: 1rem;
  }
}

.fixed-width {
  width: 10rem;
}
</style>
