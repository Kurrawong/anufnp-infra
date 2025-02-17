<script setup lang="ts">
import { Cog, ChevronRight, ChevronLeft } from "lucide-vue-next";

const props = defineProps<{ sidepanel?: boolean; contentonly?: boolean }>();
const route = useRoute();
const appConfig = useAppConfig();
const runtimeConfig = useRuntimeConfig();
const globalConfig = useGlobalConfig();
const apiEndpoint = useGetPrezAPIEndpoint();
const altEndpoints = useGetPrezAPIAltEndpoints();
const menu = appConfig.menu;
const expandSidePanel = ref(false);
const showDebugPanel = ref(false);

onBeforeMount(() => {
  if (typeof localStorage !== "undefined") {
    expandSidePanel.value = !!localStorage.getItem("expandSidePanel");
    showDebugPanel.value =
      runtimeConfig.public.prezDebug && !!localStorage.getItem("debug");
    watch(expandSidePanel, (val) =>
      localStorage.setItem("expandSidePanel", (val && "1") || ""),
    );
    watch(showDebugPanel, (val) =>
      localStorage.setItem("debug", (val && "1") || ""),
    );
  }
});
</script>

<template>
  <div class="flex flex-col min-h-screen">
    <!-- Header -->
    <header class="bg-white text-secondary h-32">
      <div
        class="container mx-auto px-4 h-full flex justify-between items-center"
      >
        <!-- Logo area -->
        <NuxtLink to="/" class="text-4xl hidden md:block">
          <slot name="logo">
            <div class="flex align-bottom">
              <img
                class="w-32 h-32"
                src="https://d1blp3aor2iuhp.cloudfront.net/images/logos/logo-ANU-black.svg"
                alt="ANU Logo"
              />
            </div>
          </slot>
        </NuxtLink>
        <h1 class="text-primary">
          First Nations Portfolio Indigenous Research Portal
        </h1>
        <!-- nav -->
      </div>
    </header>

    <!-- Navigation -->
    <div class="bg-white text-secondary border-b relative">
      <nav
        class="container mx-auto px-4 py-4 hidden md:flex space-x-12 text-lg"
      >
        <NuxtLink
          v-for="{ label, url } in menu.filter((item) => item.active !== false)"
          :to="url"
          :class="`border-b-[5px] hover:border-primary ${(url === '/' && route.path === '/') || (url !== '/' && route.path.startsWith(url)) ? 'text-primary border-primary' : 'border-transparent'}`"
          >{{ label }}</NuxtLink
        >
        <div v-if="runtimeConfig.public.prezDebug" class="!ml-auto">
          <div v-if="showDebugPanel">
            <span
              title="Toggle debug off"
              class="hover:cursor-pointer hover:text-gray-500 text-blue-400"
              @click="
                () => {
                  showDebugPanel = !showDebugPanel;
                }
              "
              ><Cog class="w-4 h-4"
            /></span>
          </div>
          <span
            v-else
            title="Toggle debug on"
            class="hover:cursor-pointer hover:text-gray-500 text-gray-300"
            @click="
              () => {
                showDebugPanel = !showDebugPanel;
              }
            "
            ><Cog class="w-4 h-4"
          /></span>
        </div>
      </nav>
    </div>

    <slot v-if="!contentonly" name="header">
      <div class="bg-white">
        <div class="container mx-auto flex flex-row">
          <div class="px-4 py-4 flex-grow">
            <slot name="breadcrumb" />
            <div class="text-3xl pb-4 pt-3">
              <slot name="header-text" />
            </div>
          </div>
          <div
            v-if="showDebugPanel"
            class="m-2 bg-gray-200 rounded-lg text-[12px] leading-[12px]"
          >
            <slot name="debug" />
          </div>
        </div>
      </div>
    </slot>
    <div v-else-if="showDebugPanel" class="bg-gray-100">
      <div class="container px-4 py-4 mx-auto">
        <slot name="debug" />
      </div>
    </div>

    <div class="container mx-auto flex-grow">
      <div v-if="sidepanel" class="grid grid-cols-4 gap-4 px-4 py-4">
        <div
          :class="
            expandSidePanel ? 'col-span-3 relative' : 'col-span-4 relative'
          "
        >
          <slot />
          <Button
            v-if="!expandSidePanel"
            title="Show sidepanel"
            variant="ghost"
            size="icon"
            class="absolute right-0 top-[-5px] pointer-events-auto"
            @click="expandSidePanel = !expandSidePanel"
          >
            <ChevronLeft class="size-4" />
          </Button>
        </div>
        <div v-if="expandSidePanel" class="relative">
          <slot name="sidepanel" />
          <Button
            title="Hide sidepanel"
            variant="ghost"
            size="icon"
            class="absolute right-0 top-[-5px] pointer-events-auto"
            @click="expandSidePanel = !expandSidePanel"
          >
            <ChevronRight class="size-4" />
          </Button>
        </div>
      </div>
      <div v-else class="px-4 py-4">
        <slot />
      </div>
    </div>

    <!-- Footer -->
    <footer class="bg-primary text-white pt-6 pb-10">
      <div
        class="border-l-[3px] border-secondary pl-4 pt-4 w-[80%] mx-auto"
      >
        <div class="flex justify-between items-center">
          <img
            class="w-32 h-32 translate-x-[-32px] bg-primary"
            src="https://d1blp3aor2iuhp.cloudfront.net/images/logos/logo-ANU.svg"
            alt="ANU Logo"
          />
          <div class="flex space-x-14 pr-14">
            <a
              href="https://www.anu.edu.au/association-of-pacific-rim-universities"
            >
              <img
                class="w-32 h-32"
                src="https://d1blp3aor2iuhp.cloudfront.net/images/logos/logo-APRU.svg"
                alt="APRU Logo"
              />
            </a>
            <a
              href="https://www.anu.edu.au/international-alliance-of-research-universities"
            >
              <img
                class="w-32 h-32"
                src="https://d1blp3aor2iuhp.cloudfront.net/images/logos/logo-IARU.svg"
                alt="IARU Logo"
            /></a>
            <a href="https://www.anu.edu.au/edx"
              ><img
                class="w-20 h-32"
                src="https://d1blp3aor2iuhp.cloudfront.net/images/logos/logo-EDX.svg"
                alt="Edx Logo"
            /></a>
            <a href="https://www.anu.edu.au/group-of-eight"
              ><img
                class="w-24 h-32"
                src="https://d1blp3aor2iuhp.cloudfront.net/images/logos/logo-GroupOfEight.svg"
                alt="group of eight logo"
            /></a>
          </div>
        </div>
        <div class="container">
          <p class="text-lg mb-2 font-bold">Acknowledgement of Country</p>
          <p class="mb-4">
            The Australian National University acknowledges, celebrates and pays
            our respects to the Ngunnawal and Ngambri people of the Canberra
            region and to all First Nations Australians on whose traditional
            lands we meet and work, and whose cultures are among the oldest
            continuing cultures in human history.
          </p>
        </div>
        <hr class="mb-4" />
        <div class="container text-center">
          <a
            class="text-primary-foreground border-dashed border-b-[2px] hover:border-solid"
            href="https://www.anu.edu.au/contact"
            >Contact ANU</a
          >
          |
          <a
            class="text-primary-foreground border-dashed border-b-[2px] hover:border-solid"
            href="https://www.anu.edu.au/copyright"
            >Copyright</a
          >
          |
          <a
            class="text-primary-foreground border-dashed border-b-[2px] hover:border-solid"
            href="https://www.anu.edu.au/disclaimer"
            >Disclaimer</a
          >
          |
          <a
            class="text-primary-foreground border-dashed border-b-[2px] hover:border-solid"
            href="https://www.anu.edu.au/privacy"
            >Privacy</a
          >
          |
          <a
            class="text-primary-foreground border-dashed border-b-[2px] hover:border-solid"
            href="https://www.anu.edu.au/freedom-of-information"
            >Freedom of Information</a
          >
          <p class="mt-4">
            +61 2 6125 5111 | The Australian National University, Canberra TEQSA
            Provider ID: PRV12002 (Australian University) | CRICOS Provider
            Code: 00120C | ABN: 52 234 063 906
          </p>
        </div>
        <div class="container text-center">
          <small>PrezUI {{ runtimeConfig.app.version }}</small> |
          <small
            ><a v-if="globalConfig?.version" :href="apiEndpoint" target="_new"
              >PrezAPI {{ globalConfig?.version }}</a
            ></small
          >
        </div>
        <div
          v-if="
            apiEndpoint != runtimeConfig.public.prezApiEndpoint &&
            !altEndpoints.find((e) => e.endpoint == apiEndpoint)
          "
        >
          <em
            ><small>custom override API endpoint {{ apiEndpoint }}</small></em
          >
        </div>
        <ul
          v-if="altEndpoints.length > 0"
          class="flex space-x-1 text-sm text-gray-400 justify-center [&>li:not(:last-child)]:after:content-['|'] [&>li:not(:last-child)]:after:mx-2"
        >
          <li
            class="hover:cursor-pointer"
            v-for="{ endpoint, name } of [
              {
                name: 'Default',
                endpoint: runtimeConfig.public.prezApiEndpoint,
              },
              ...altEndpoints,
            ]"
          >
            <a
              :class="apiEndpoint == endpoint ? '!text-yellow-200' : ''"
              :href="`/?_api=${endpoint}`"
              >{{ name }}</a
            >
          </li>
        </ul>
      </div>
    </footer>
  </div>
</template>
