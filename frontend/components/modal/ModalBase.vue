<!-- SPDX-License-Identifier: AGPL-3.0-or-later -->
<template>
  <Dialog class="relative z-50" :open="modalIsOpen">
    <DialogBackdrop
      className="fixed inset-0 bg-layer-0/95 dark:bg-layer-0/95"
    />
    <div
      @click="closeModal()"
      @keydown.enter="closeModal()"
      class="cursor-pointer"
      :class="{
        'fixed top-0 z-10 flex h-screen w-full flex-col items-center overflow-hidden':
          imageModal,
        'fixed inset-0 flex w-screen items-center justify-center': !imageModal,
      }"
    >
      <DialogPanel
        id="search-modal"
        :class="{
          'flex flex-col items-center': imageModal,
          'card-style-base container h-full w-full max-w-4xl cursor-default overflow-y-auto bg-layer-0 p-5 pl-6 text-primary-text md:h-auto':
            !imageModal,
        }"
      >
        <button
          v-if="imageModal"
          @click="closeModal()"
          id="close-search-modal"
          class="focus-brand absolute right-0 mr-24 mt-8 rounded-full p-1 text-distinct-text hover:text-primary-text"
          :aria-label="
            $t ? $t('i18n.components.modal_base.close_modal_aria_label') : ''
          "
        >
          <Icon class="h-10 w-10" :name="IconMap.CIRCLE_X_FILL" />
        </button>
        <div v-else class="relative">
          <button
            @click="closeModal()"
            class="focus-brand absolute right-0 rounded-full p-1 text-distinct-text hover:text-primary-text"
          >
            <Icon class="h-10 w-10" :name="IconMap.CIRCLE_X_FILL" />
          </button>
        </div>
        <div
          v-if="imageModal"
          @click="closeModal()"
          @keypress.esc="closeModal()"
          tabindex="0"
          role="button"
          class="focus-brand flex flex-col items-center justify-center"
          :aria-label="
            $t ? $t('i18n.components.modal_base.close_modal_aria_label') : ''
          "
        >
          <slot />
        </div>
        <div v-else>
          <slot />
        </div>
      </DialogPanel>
    </div>
  </Dialog>
</template>

<script setup lang="ts">
import { Dialog, DialogPanel } from "@headlessui/vue";
import { useRoute } from "vue-router";

import { IconMap } from "~/types/icon-map";

const props = defineProps<{
  imageModal?: boolean;
  modalName: string;
}>();

const emit = defineEmits(["closeModal"]);

const route = useRoute();
const modals = useModals();
const modalName = props.modalName;
const modalIsOpen = ref(false);

onMounted(() => {
  // Ensure modals.modals[modalName] is defined. If so, save isOpen in store.
  if (modals.modals[modalName]) {
    modalIsOpen.value = modals.modals[modalName].isOpen;
  }
});

// Watch the reactive modalIsOpen ref and check if modalName exists in modals.modals.
watch(
  () => modals.modals[modalName]?.isOpen,
  (newVal) => {
    if (newVal !== undefined) {
      modalIsOpen.value = newVal;
    }
  }
);

// Tell any interested observers that the closeModal event has happened.
// The interested observer(s) can respond or do cleanup specific to their needs.
const closeModal = () => {
  emit("closeModal");
  modals.closeModal(modalName);
};

// Check if the user is navigating to another resource.
// If a modal exists, close it.
watch(route, () => {
  if (modals.modals[modalName]) {
    closeModal();
  }
});
</script>
