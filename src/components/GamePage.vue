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

      setGravity(1600);

      // Load the sprite
      loadSprite("bean", "./sprites/bean.png").catch((err) => {
        console.error("Error loading sprite:", err);
      });
      loadSprite("grass", "./sprites/grass.png").catch((err) => {
        console.error("Error loading grass sprite:", err);
      });

      // Add player game object
      const player = add([
        sprite("bean"),
        pos(center()),
        area(),
        // body() component gives the ability to respond to gravity
        body(),
      ]);

      const SPEED = 320;
      onKeyDown("left", () => player.move(-SPEED, 0));
      onKeyDown("right", () => player.move(SPEED, 0));
      onKeyDown("up", () => player.jump());
      onKeyDown("down", () => player.move(0, SPEED));
      onKeyDown("a", () => player.move(-SPEED, 0));
      onKeyDown("d", () => player.move(SPEED, 0));
      onKeyDown("w", () => player.jump());
      onKeyDown("s", () => player.move(0, SPEED));
      // Jump higher if space is held
      onKeyDown("w", () => {
        if (!player.isGrounded() && player.vel.y < 0) {
          player.vel.y -= dt() * 600;
        }
      });
      // Jump higher if space is held
      onKeyDown("up", () => {
        if (!player.isGrounded() && player.vel.y < 0) {
          player.vel.y -= dt() * 600;
        }
      });
      // Accelerate falling when player holding down arrow key
      onKeyDown("down", () => {
        if (!player.isGrounded()) {
          player.vel.y += dt() * 1200;
        }
      });

      // Add a platform to hold the player
      const grassWidth = 64; // Assuming the grass sprite width is 64 pixels
      for (let x = 0; x <= 1280; x += grassWidth) {
        add([
          sprite("grass"), // Use the grass sprite
          area(),
          pos(x, 550), // Place at x, 600
          body({ isStatic: true }), // Make it static
        ]);
      }

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
