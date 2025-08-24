<template>
  <div>
    <NuxtRouteAnnouncer>
      <template #default="{ route }">
        <div>
          You are now on the <strong>{{ route.name }}</strong> page.
        </div>
      </template>
    </NuxtRouteAnnouncer>
    <NuxtWelcome />
  </div>
</template>
