<template>
  <q-layout class="profile-layout" view="lhh LpR lff">
    <div @click="logoutUser" class="logout-btn">
      <img
        src="/status-override/images/icons/logout-icon.svg"
        alt=""
        srcset=""
      />
    </div>
    <header class="profile-header">
      <q-toolbar class="header-toolbar flex justify-end q-ml-sm">
        <img
          class="q-mt-md"
          src="/status-override/images/khafji-logo2.png"
          alt="logo"
        />
      </q-toolbar>
      <div class="profile-container row justify-center">
        <div class="profile-img-wrapper col-8">
          <img class="fit" :src="profilePicture" alt="profile-image" />
        </div>
      </div>
    </header>
    <q-page-container>
      <q-page class="user-data min-height-auto q-mt-lg">
        <div class="items-end q-mt-md flex column">
          <span class="text-black user-data_name q-mt-sm">
            {{ fullName }}
          </span>
          <span class="profile-work user-data_title q-mt-sm">
            {{ title }}
          </span>
          <div class="user-data_office row justify-center items-center">
            <span>office : </span>
            <span> {{ office }} </span>
          </div>
        </div>
        <q-card flat class="status-card q-py-lg">
          <div class="status-card_toggle row justify-end">
            <span> تغير الحالة</span>
            <q-toggle class="toggle-status" v-model="toggleValue" />
          </div>

          <div>
            <q-tabs
              v-model="currentStatus"
              indicator-color="transparent"
              :active-bg-color="activeColor"
              class="status-tabs text-black q-my-lg"
            >
              <q-tab
                class="col-5 q-mb-md"
                v-for="status in AllStatus"
                :key="status.id"
                :name="status.id"
                :icon="statusIcon(status.id)"
                :label="status.name_Ar"
                :disable="toggleValue"
                no-caps
                @click="handleChange"
                dark
              />
            </q-tabs>
          </div>
        </q-card>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { computed, onMounted, ref, watch } from 'vue';
import {
  colorNames,
  shadowValues,
  iconNames,
} from 'src/services/static-lookups';
import { getCssVar } from 'quasar';
import { useAppStore } from '../store';
import { useAuthStore } from 'src/modules/auth/store/index';
import { storeToRefs } from 'pinia';
import { notify } from 'src/boot/plugins/notify';
import { useI18n } from 'vue-i18n';
import storage from 'src/services/storage';

// import { useRouter } from 'vue-router';
// const router = useRouter();
const { t } = useI18n();

//todo
const AuthStore = useAuthStore();
const { logoutUser } = AuthStore;

const appStore = useAppStore();
const {
  profileId,
  fullName,
  title,
  office,
  profilePicture,
  currentStatus,
  toggleValue,
  AllStatus,
} = storeToRefs(appStore);
const {
  getManagerProfileByEmail,
  updateManagerProfileStatusByEmail,
  updateGetProfileStatusFromExchange,
} = appStore;

const statusIds = computed(() => AllStatus.value?.map((status) => status.id));

// const imageStyle = computed(() => {
//   const statusColors = statusIds.value?.reduce((colors, id, index) => {
//     colors[id] = colorNames[index] || ''; // Assign color name based on index, or empty string if index exceeds available color names
//     return colors;
//   }, {});

const imageShadows = statusIds.value?.reduce((shadows, id, index) => {
  shadows[id] = shadowValues[index] || ''; // Assign shadow value based on index, or empty string if index exceeds available shadow values
  return shadows;
}, {});

const activeColor = computed(() => {
  const statusColors = statusIds.value?.reduce((colors, id, index) => {
    colors[id] = colorNames[index] || ''; // Assign color name based on index, or empty string if index exceeds available color names
    return colors;
  }, {});

  return statusColors?.[currentStatus.value];
});
// const whIcon = ref('');
const statusIcon = (statusId) => {
  const statusIconMapping = statusIds.value?.reduce((result, id, index) => {
    result[id] = iconNames[index] || ''; // Assign file name based on index, or empty string if index exceeds available icon names
    return result;
  }, {});
  return `img:/status-override/images/icons/${statusIconMapping[statusId]}.svg`;
};

const handleChange = (e) => {
  if (currentStatus.value) {
    updateManagerProfileStatusByEmail(profileId.value, currentStatus.value);
    // const statusIcon = (statusId) => {
    //   const statusIconMapping = statusIds.value?.reduce((result, id, index) => {
    //     result[id] = iconNames[index] || ''; // Assign file name based on index, or empty string if index exceeds available icon names
    //     return result;
    //   }, {});

    //   return `img:/status-override/images/icons/${statusIconMapping[statusId]}`;
    // };
  } else {
    notify('error', t('app.choose'));
  }
};

watch(
  () => toggleValue.value,
  (val) => {
    if (val) {
      currentStatus.value = null;
    }
    updateGetProfileStatusFromExchange(profileId.value, toggleValue.value);
  }
);

onMounted(() => {
  const email = storage.getProfile().email;
  getManagerProfileByEmail(email);
});
</script>

<style lang="scss">
@import 'src/css/profile.scss';
</style>
