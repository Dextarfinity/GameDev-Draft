<template>
  <div id="game-container"></div>
</template>

<script>
import { onMounted } from "vue";
import kaboom from "kaboom";

export default {
  name: "App",
  setup() {
    onMounted(() => {
      // Initialize Kaboom once the component is mounted
      kaboom();

      // Load the sprite
      loadSprite("bean", "./sprites/bean.png").catch((err) => {
        console.error("Error loading sprite:", err);
      });

      const player = add([sprite("bean"), pos(center())]);

      const SPEED = 320;
      onKeyDown("left", () => player.move(-SPEED, 0));
      onKeyDown("right", () => player.move(SPEED, 0));
      onKeyDown("up", () => player.move(0, -SPEED));
      onKeyDown("down", () => player.move(0, SPEED));

      onClick(() => player.moveTo(mousePos()));

      add([text("Press arrow keys", { width: width() / 2 }), pos(12, 12)]);
    });
  },
};
</script>

<style scoped>
#game-container {
  width: 100%;
  height: 100vh;
}
</style>
