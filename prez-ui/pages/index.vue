<template>
  <NuxtLayout contentonly>
    <template #default>
      <div class="container mb-10">
        <h1 class="text-2xl mt-8">
          First Nations Portfolio Indigenous Research Portal
        </h1>
      </div>
      <div class="text-lg">
        <p class="mb-4">
          Use this portal to discover Indigenous research data held by the ANU.
        </p>
        <p class="mb-8">
          The catalogs hosted here expose historical research data from across
          the university.
        </p>
        <NuxtLink to="/catalogs" class="btn btn-primary px-4 py-2 rounded shadow bg-primary text-white hover:text-black hover:bg-secondary transition">
          Browse the Portal &#8594;
        </NuxtLink>
      </div>
    </template>
  </NuxtLayout>
</template>
