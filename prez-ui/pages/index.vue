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
        <p class="mb-4">
          The catalogs hosted here expose historical research data from across
          the university.
        </p>
        <p class="mb-4">
          The data is provided in RDF Format, making it highly interoperable and
          discoverable.
        </p>
        <NuxtLink to="/catalogs" class="btn btn-primary mt-4">
          View the catalogs
        </NuxtLink>
      </div>
    </template>
  </NuxtLayout>
</template>
