<template>
  <div class="w-full flex">
    <div class="flex-1 gap-4 flex flex-col items-start">
      <UFormGroup
        label="Nome"
        name="nome"
        help="diferente do nome de usuário(apelido), esse é o teu nome"
      >
        <UInput class="w-full max-w-xs" v-model="state.name" />
      </UFormGroup>

      <UFormGroup class="w-full" label="Bio" name="bio">
        <UTextarea resize class="w-full max-w-xs" v-model="state.bio" />
      </UFormGroup>

      <UButton
        type="submit"
        @click="onSubmit"
        loading-icon="i-carbon-circle-dash"
        :loading="isLoading"
      >
        <span class="text-white font-normal">Atualizar</span>
      </UButton>
    </div>
    <div class="flex flex-col items-center gap-2">
      <div class="w-60 h-60 rounded-full overflow-hidden">
        <img class="w-full" :src="AvatarImage" alt="" />
      </div>
      <UInput
        :loading="isFileUpdating"
        type="file"
        size="sm"
        icon="i-heroicons-folder"
        loading-icon="i-carbon-circle-dash"
        @change="updateImage($event)"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import AvatarImage from "@/public/avatar.jpg";
import { useAuthStore } from "@/store";
import { useUser, useFile } from "@/composables";

definePageMeta({
  alias: "/settings",
  layout: "settings",
});

const toast = useToast();
const { user } = useAuthStore();
const { updateUser } = useUser();
const { updateUserStore } = useAuthStore();
const { imageTo64 } = useFile();
const isLoading = ref(false);
const isFileUpdating = ref(false);
const state = ref({
  id: "",
  name: "",
  bio: "",
});
async function updateImage(e: File[]) {
  isFileUpdating.value = true;
  const imageUrl = await imageTo64(e[0]);
  const response = await updateUser({ photo: imageUrl });
  if (response) {
    isFileUpdating.value = false;
    useToast().add({ title: response?.message, timeout: 5000, color: "red" });
  }
}

async function onSubmit() {
  isLoading.value = true;
  const response = await updateUser(state.value);
  if (response?.statusCode == 200) {
    const osSave = await updateUserStore(state.value);
    if (osSave) {
      isLoading.value = false;
      toast.add({ title: "Usuário atualizado", timeout: 2000 });
    }
  } else {
    console.log(response);
  }
}

watchEffect(() => {
  state.value = { ...user };
  delete state.value.id;
});
</script>