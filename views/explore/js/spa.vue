<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  window.spa = window.spa || {};

  window.spa.initModule = function ($container) {
    window.spa.shell.initModule($container);
  };

  return window.spa;
}
</script>
