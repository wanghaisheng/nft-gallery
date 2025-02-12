<template>
  <div>
    <div>
      <section class="flex controls">
        <NeoButton class="left" @click="toOnborading">
          <NeoIcon icon="arrow-left" class="mr-1" />
          {{ $t('massmint.backToOnbaording') }}
        </NeoButton>
        <div class="dropdown-container">
          <div>
            <p class="mb-4">{{ $t('massmint.chooseCollection') }}</p>
            <ChooseCollectionDropdown
              :preselected="preselectedCollectionId"
              @selectedCollection="onCollectionSelected" />
          </div>
        </div>
      </section>

      <section class="border k-shadow mt-7">
        <UploadMediaZip
          :disabled="!selectedCollection"
          @zipLoaded="onMediaZipLoaded" />
        <UploadDescription
          :disabled="!mediaLoaded"
          @fileLoaded="onDescriptionLoaded" />
        <OverviewTable
          :disabled="!mediaLoaded"
          :nfts="NFTS"
          @openSideBarWith="openSideBarWith"
          @delete="openDeleteModalWith" />
      </section>
    </div>
    <EditPanel
      :nft="nftBeingEdited"
      :open="sideBarOpen"
      @close="sideBarOpen = false"
      @save="updateNFT" />
    <div class="mt-6 flex justify-center w-full">
      <NeoButton
        class="flex flex-grow limit-width"
        variant="k-accent"
        size="large"
        :disabled="!mediaLoaded"
        @click="openReviewModal">
        <span class="is-size-5"
          >{{ $t('massmint.mintNFTs') }}
          <span v-if="numOfValidNFTs" class="font-bold">
            ({{ numOfValidNFTs }})
          </span>
        </span>
      </NeoButton>
    </div>
    <DeleteModal
      v-if="nftInDeleteModal"
      v-model="deleteModalOpen"
      :nft="nftInDeleteModal"
      @close="closeDeleteModal"
      @delete="deleteNFT" />
    <MissingInfoModal
      v-model="missingInfoModalOpen"
      :num-missing-names="numberOfMissingNames"
      :num-missing-descriptions="numberOfMissingDescriptions"
      :num-missing-prices="numberOfMissingPrices"
      @close="missingInfoModalOpen = false" />
    <ReviewModal
      v-model="overViewModalOpen"
      :num-missing-descriptions="numberOfMissingDescriptions"
      :num-missing-prices="numberOfMissingPrices"
      :num-nfts="Object.keys(NFTS).length"
      @close="overViewModalOpen = false"
      @mint="startMint" />
    <MintingModal
      v-model="mintModalOpen"
      :loading="isMinting"
      @close="mintModalOpen = false" />
    <MobileDisclaimerModal
      v-model="MobileDisclaimerModalOpen"
      @continue="MobileDisclaimerModalOpen = false"
      @leave="back" />
  </div>
</template>

<script setup lang="ts">
import { NeoButton, NeoIcon } from '@kodadot1/brick'
import { usePreferencesStore } from '@/stores/preferences'
import UploadMediaZip from './uploadCompressedMedia/UploadCompressedMedia.vue'
import UploadDescription from './uploadDescription/UploadDescription.vue'
import OverviewTable from './OverviewTable.vue'
import ChooseCollectionDropdown from '@/components/common/ChooseCollectionDropdown.vue'
import EditPanel from './EditPanel.vue'
import { NFT, NFTToMint } from './types'
const { width } = useWindowSize()
const { back } = useRouter()

import {
  DeleteModal,
  MintingModal,
  MissingInfoModal,
  MobileDisclaimerModal,
  ReviewModal,
} from './modals'
import { MintedCollection } from '@/composables/transaction/types'
import { notificationTypes, showNotification } from '@/utils/notification'
import { useMassMint } from '@/composables/massmint/useMassMint'
import { Entry } from '@/composables/massmint/parsers/common'
import { FileObject } from '@/composables/massmint/useZipValidator'

const preferencesStore = usePreferencesStore()
const { $consola, $i18n } = useNuxtApp()
const router = useRouter()
const { urlPrefix } = usePrefix()
const { selectedCollection, preselectedCollectionId, onCollectionSelected } =
  useCollectionDropdown()

const NFTS = ref<{ [nftId: string]: NFT }>({})
const mediaLoaded = ref(false)

const nftBeingEdited = ref<NFT>()
const nftInDeleteModal = ref<NFT>()
const sideBarOpen = ref(false)
const deleteModalOpen = ref(false)
const missingInfoModalOpen = ref(false)
const overViewModalOpen = ref(false)
const mintModalOpen = ref(false)
const MobileDisclaimerModalOpen = ref(false)
const smallerThenDesktop = computed(() => width.value < 1024)

const isMinting = ref(false)
const mintStatus = ref('')

const numberOfMissingNames = computed(
  () => Object.values(NFTS.value).filter((nft) => !nft.name).length,
)

const numOfValidNFTs = computed(
  () => Object.values(NFTS.value).length - numberOfMissingNames.value,
)
const numberOfMissingDescriptions = computed(
  () => Object.values(NFTS.value).filter((nft) => !nft.description).length,
)

const numberOfMissingPrices = computed(
  () => Object.values(NFTS.value).filter((nft) => !nft.price).length,
)

const openSideBarWith = (nft: NFT) => {
  nftBeingEdited.value = nft
  sideBarOpen.value = true
}
const openDeleteModalWith = (nft: NFT) => {
  nftInDeleteModal.value = nft
  deleteModalOpen.value = true
}
const closeDeleteModal = () => {
  deleteModalOpen.value = false
  nftInDeleteModal.value = undefined
}

const openReviewModal = () => {
  if (numberOfMissingNames.value > 0) {
    missingInfoModalOpen.value = true
    return
  }
  overViewModalOpen.value = true
}

const startMint = () => {
  overViewModalOpen.value = false
  mintModalOpen.value = true
  isMinting.value = true

  const { isLoading, status, collectionUpdated, isError } = useMassMint(
    Object.values(NFTS.value) as NFTToMint[],
    selectedCollection.value as MintedCollection,
  )

  watch(
    [isLoading, status, collectionUpdated, isError],
    ([isLoadingV, statusV], [isLoadingOldV]) => {
      isMinting.value = isLoadingV
      mintStatus.value = statusV

      if (isLoadingOldV && !isLoadingV) {
        mintModalOpen.value = false
        if (!isError.value && statusV !== TransactionStatus.Sign) {
          showNotification(
            $i18n.t('massmint.continueToCollectionPage'),
            notificationTypes.success,
          )
        }
      }

      //redirect to collection page when collection is updated
      if (collectionUpdated.value) {
        navigateTo(
          `/${urlPrefix.value}/collection/${selectedCollection.value?.id}`,
        )
      }
    },
  )
}

const updateNFT = (nft: NFT) => {
  NFTS.value[nft.id] = nft
}

const deleteNFT = (nft?: NFT) => {
  if (!nft) {
    return
  }
  NFTS.value = Object.values(NFTS.value)
    .filter((n) => n.id !== nft.id)
    .map((nft, i) => ({ ...nft, id: i + 1 }))
    .reduce((acc, nft) => ({ ...acc, [nft.id]: nft }), {})

  closeDeleteModal()
}

const toOnborading = () => {
  preferencesStore.setVisitedOnboarding(false)
  router
    .replace({
      path: `/${urlPrefix.value}/massmint/onboarding`,
    })
    .catch($consola.warn)
}

const onMediaZipLoaded = ({ validFiles }: { validFiles: FileObject[] }) => {
  NFTS.value = validFiles
    .map((file, i) => ({ ...file, id: i + 1 }))
    .reduce((acc, nft) => ({ ...acc, [nft.id]: nft }), {})
  mediaLoaded.value = true
}
const onDescriptionLoaded = (entries: Record<string, Entry>) => {
  // create a map of nft filename to id
  const nftFileNameToId = Object.values(NFTS.value).reduce(
    (acc, nft) => ({ ...acc, [nft.file.name]: nft.id }),
    {},
  )
  Object.values(entries).forEach((entry) => {
    if (!entry.valid) {
      return
    }
    const nftId = nftFileNameToId[entry.file]
    if (!nftId) {
      return
    }
    // eslint-disable-next-line @typescript-eslint/no-unused-vars
    const { file: _, ...restOfEntry } = entry
    NFTS.value[nftId] = {
      ...NFTS.value[nftId],
      ...restOfEntry,
    }
  })
}

onMounted(() => {
  MobileDisclaimerModalOpen.value = smallerThenDesktop.value
})
</script>
<style lang="scss" scoped>
@import '@/assets/styles/abstracts/variables.scss';

.controls {
  justify-content: center;
  align-items: flex-end;

  .left {
    position: absolute;
    left: 2.5rem;
  }
}

@include touch {
  .controls {
    flex-direction: column;
    gap: 2rem;
    align-items: flex-start;

    .dropdown-container {
      align-self: center;
    }
    .left {
      position: unset;
    }
  }
}

.limit-width {
  max-width: 45rem;
}
</style>

<style lang="scss">
.tab-nft {
  .explore-tabs-button {
    border-left: 0 !important;
    border-right: 0 !important;
  }
}
</style>
