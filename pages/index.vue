<template>
  <div class="flex flex-col justify-center items-center h-screen w-full">
    <nuxt-link to="/inspect">
      <button class="h-[50px] px-8 bg-teal-700 text-white rounded-sm">
        Start Inspection
      </button>
    </nuxt-link>
  </div>
</template>

<script>
export default {
  name: "IndexPage",
};
</script>
