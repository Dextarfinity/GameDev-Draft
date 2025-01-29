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
      const FLOOR_HEIGHT = 36;
      const JUMP_FORCE = 900;
      const SPEED = 480;

      // initialize context
      kaboom();

      setBackground(141, 183, 255);

      scene("game", () => {
        // define gravity
        setGravity(2400);

        // add a game object to screen

        // Loading a multi-frame sprite
        loadSprite("dino", "sprites/dinosaur.png", {
          // The image contains 9 frames layed out horizontally, slice it into individual frames
          sliceX: 9,
          // Define animations
          anims: {
            idle: {
              // Starts from frame 0, ends at frame 3
              from: 0,
              to: 3,
              // Frame per second
              speed: 5,
              loop: true,
            },
            run: {
              from: 4,
              to: 7,
              speed: 10,
              loop: true,
            },
            // This animation only has 1 frame
            jump: 8,
          },
        });

        // Add our player character
        const player = add([
          sprite("dino"),
          pos(80, 40),
          anchor("center"),
          area(),
          body(),
          scale(4), // Scale the dino sprite to twice its size
        ]);

        // .play is provided by sprite() component, it starts playing the specified animation (the animation information of "idle" is defined above in loadSprite)
        player.play("idle");

        // Add a platform
        add([
          rect(width(), 24),
          area(),
          outline(1),
          pos(0, height() - 24),
          body({ isStatic: true }),
        ]);

        // Switch to "idle" or "run" animation when player hits ground
        player.onGround(() => {
          if (!isKeyDown("left") && !isKeyDown("right")) {
            player.play("idle");
          } else {
            player.play("run");
          }
        });

        player.onAnimEnd((anim) => {
          if (anim === "idle") {
            // You can also register an event that runs when certain anim ends
          }
        });

        onKeyPress("space", () => {
          if (player.isGrounded()) {
            player.jump(JUMP_FORCE);
            player.play("jump");
          }
        });

        onKeyDown("left", () => {
          player.move(-SPEED, 0);
          player.flipX = true;
          // .play() will reset to the first frame of the anim, so we want to make sure it only runs when the current animation is not "run"
          if (player.isGrounded() && player.curAnim() !== "run") {
            player.play("run");
          }
        });

        onKeyDown("right", () => {
          player.move(SPEED, 0);
          player.flipX = false;
          if (player.isGrounded() && player.curAnim() !== "run") {
            player.play("run");
          }
        });
        ["left", "right"].forEach((key) => {
          onKeyRelease(key, () => {
            // Only reset to "idle" if player is not holding any of these keys
            if (player.isGrounded() && !isKeyDown("left") && !isKeyDown("right")) {
              player.play("idle");
            }
          });
        });

        const getInfo = () =>
          `
        Anim: ${player.curAnim()}
        Frame: ${player.frame}
        `.trim();

        // Add some text to show the current animation
        const label = add([text(getInfo(), { size: 12 }), color(0, 0, 0), pos(4)]);

        label.onUpdate(() => {
          label.text = getInfo();
        });

        // floor
        add([
          rect(width(), FLOOR_HEIGHT),
          outline(4),
          pos(0, height()),
          anchor("botleft"),
          area(),
          body({ isStatic: true }),
          color(132, 101, 236),
        ]);

        function jump() {
          if (player.isGrounded()) {
            player.jump(JUMP_FORCE);
          }
        }

        // jump when user press space
        onKeyPress("space", jump);
        onClick(jump);

        function spawnTree() {
          // add tree obj
          add([
            rect(48, rand(32, 96)),
            area(),
            outline(4),
            pos(width(), height() - FLOOR_HEIGHT),
            anchor("botleft"),
            color(238, 143, 203),
            move(LEFT, SPEED),
            offscreen({ destroy: true }),
            "tree",
          ]);

          // wait a random amount of time to spawn next tree
          wait(rand(0.5, 1.5), spawnTree);
        }

        // start spawning trees
        spawnTree();

        // lose if player collides with any game obj with tag "tree"
        player.onCollide("tree", () => {
          // go to "lose" scene and pass the score
          go("lose", score);
          burp();
          addKaboom(player.pos);
        });

        // keep track of score
        let score = 0;

        const scoreLabel = add([text(score), pos(24, 24)]);

        // increment score every frame
        onUpdate(() => {
          score++;
          scoreLabel.text = score;
        });
      });

      scene("lose", (score) => {
        add([
          sprite("dino"),
          pos(width() / 2, height() / 2 - 64),
          scale(4),
          anchor("center"),
        ]);

        // display score
        add([
          text(score),
          pos(width() / 2, height() / 2 + 64),
          scale(2),
          anchor("center"),
        ]);

        // go back to game with space is pressed
        onKeyPress("space", () => go("game"));
        onClick(() => go("game"));
      });

      go("game");
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
