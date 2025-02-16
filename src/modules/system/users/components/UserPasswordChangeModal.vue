<script lang="ts" setup>
import { VButton, VModal, VSpace } from "@halo-dev/components";
import SubmitButton from "@/components/button/SubmitButton.vue";
import { inject, ref, watch } from "vue";
import type { User } from "@halo-dev/api-client";
import { apiClient } from "@/utils/api-client";
import cloneDeep from "lodash.clonedeep";
import { reset } from "@formkit/core";
import { setFocus } from "@/formkit/utils/focus";

const props = withDefaults(
  defineProps<{
    visible: boolean;
    user?: User;
  }>(),
  {
    visible: false,
    user: undefined,
  }
);

const emit = defineEmits<{
  (event: "update:visible", visible: boolean): void;
  (event: "close"): void;
}>();

const currentUser = inject<User>("currentUser");

interface PasswordChangeFormState {
  password: string;
  password_confirm?: string;
}

const initialFormState: PasswordChangeFormState = {
  password: "",
  password_confirm: "",
};

const formState = ref<PasswordChangeFormState>(cloneDeep(initialFormState));
const saving = ref(false);

watch(
  () => props.visible,
  (visible) => {
    if (visible) {
      setFocus("passwordInput");
    } else {
      handleResetForm();
    }
  }
);

const onVisibleChange = (visible: boolean) => {
  emit("update:visible", visible);
  if (!visible) {
    emit("close");
  }
};

const handleResetForm = () => {
  formState.value = cloneDeep(initialFormState);
  reset("password-form");
};

const handleChangePassword = async () => {
  try {
    saving.value = true;

    const changePasswordRequest = cloneDeep(formState.value);
    delete changePasswordRequest.password_confirm;

    if (props.user?.metadata.name === currentUser?.metadata.name) {
      await apiClient.user.changePassword({
        name: "-",
        changePasswordRequest,
      });
    } else {
      await apiClient.user.changePassword({
        name: props.user?.metadata.name || "",
        changePasswordRequest,
      });
    }

    onVisibleChange(false);
  } catch (e) {
    console.error(e);
  } finally {
    saving.value = false;
  }
};
</script>

<template>
  <VModal
    :visible="visible"
    :width="500"
    title="密码修改"
    @update:visible="onVisibleChange"
  >
    <FormKit
      id="password-form"
      v-model="formState"
      name="password-form"
      :actions="false"
      type="form"
      :config="{ validationVisibility: 'submit' }"
      @submit="handleChangePassword"
    >
      <FormKit
        id="passwordInput"
        label="新密码"
        name="password"
        type="password"
        validation="required"
      ></FormKit>
      <FormKit
        label="确认密码"
        name="password_confirm"
        type="password"
        validation="required|confirm"
      ></FormKit>
    </FormKit>
    <template #footer>
      <VSpace>
        <SubmitButton
          v-if="visible"
          :loading="saving"
          type="secondary"
          @submit="$formkit.submit('password-form')"
        >
        </SubmitButton>
        <VButton @click="onVisibleChange(false)">取消 Esc</VButton>
      </VSpace>
    </template>
  </VModal>
</template>
