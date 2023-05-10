<script lang="ts">
  const genshinUrl = "https://api.genshin.dev/characters/";

  const lolUrlPic = "http://ddragon.leagueoflegends.com/cdn/13.9.1/img/champion/";
  const lolUrl = "http://ddragon.leagueoflegends.com/cdn/13.9.1/data/en_US/champion.json";
  let arr: [string, string][] = [];
  let champions = [];
  let unique = {};

  $: isLol = false;
  $: win = false;
  let ready = false;
  let flippable = true;

  let currEls: [string, HTMLElement][] = [];
  let completedEls: HTMLElement[] = [];

  // onMount(() => {
  //   fetch(genshinUrl, { method: "GET" })
  //     .then(response => response.json())
  //     .then(data => {
  //       champions = data;
  //       ready = true;
  //     });
  // });

  let width = 4;
  $ : max = width * width;

  $: if (win) setTimeout(() => alert("WIN"),500)

  $ : {
    if (ready) {
      arr = [];
      currEls = [];
      completedEls = [];
      unique = {};
      flippable = true;
      for (let i = 0; i < max / 2; i++) {
        if (isLol) {
          arr.push([champions[i]?.id, lolUrlPic + champions[i].id + ".png"]);
        } else {
          arr.push([champions[i], genshinUrl + champions[i] + "/" + "icon-big"]);
        }
        arr.push(arr[arr.length - 1]);
        arr = shuffle(arr);
      }
    }
  }

  $ : fetch(isLol ? lolUrl : genshinUrl, {method: "GET"})
    .then(response => response.json())
    .then(data => {
      champions = shuffle(isLol ? Object.values(data.data) : data.filter(el => !el.includes("traveler")))
      ready = true
    })


  function shuffle(array: [string, string][]) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  }

  function flipCard(event: MouseEvent) {
    if (!flippable) return;
    let a = <HTMLElement>event.currentTarget;
    if (currEls.length == 1 && currEls[0].includes(a)) return;
    currEls.push([a.dataset.name, a]);
    a.classList.add("rotate");
    if (currEls.length == 2) {
      if (currEls[0][0] == currEls[1][0]) {
        completedEls.push(currEls[0][1], currEls[1][1]);
        flippable = true
        if (completedEls.length == max) {
          win = true
        }
      } else {
        currEls.forEach(el => {
          setTimeout(() => {
            el[1].classList.remove("rotate");
            flippable = true
          }, 500);
        });
        flippable = false;
      }
      currEls = [];
    }
  }

  function swapGame() {
    isLol = !isLol;
    ready = false
  }
</script>

<div class="min-h-screen body-bg select-none overflow-hidden">
  <main class="text-white rounded-lg main-bg max-w-5xl mx-auto flex flex-col text-center align-middle p-8 mt-5">
    <section class="flex flex-col items-center">
      <h3 class="font-bold text-2xl text-center ">{isLol ? "League of legends" : "Genshin Impact"} Memory</h3>
      <input id="width-slider" type="range" min="4" step="2" max="8" bind:value={width}>
      <label for="width-slider">{width}</label>
    </section>

    <section>
    {#if ready}
      <div class="grid gap-10 justify-items-center place-content-center" style="grid-template-columns: repeat({width}, minmax(0, 1fr)">
        {#each arr as el}
          {#key unique}
            <div class="card-container relative">
              <div class="card flex rounded" on:mouseup={flipCard} data-name={el[0]}>
                <div class="face card-bg absolute flex justify-center align-items rounded-xl">
                </div>
                <div class="face rotate card-bg absolute flex justify-center align-items rounded-xl">
                  <img class="rounded" src={el[1]} alt="">
                </div>
              </div>
            </div>
          {/key}
        {/each}
      </div>
    {:else}
      <div>LOADING...</div>
    {/if}
    </section>
    <section>
      <button class="mt-5 py-3 px-4 card-bg rounded shadow-sm font-bold" on:click={swapGame}>SWAP GAME</button>
    </section>

  </main>
</div>

<style>
    .body-bg {
        background-color: #404258;
    }

    .main-bg {
        background-color: #474E68;
    }

    .card-bg {
        background-color: #50577A;
    }

    .card {
        width: 100%;
        height: 100%;
        transform-style: preserve-3d;
        transition: all 0.5s ease;
    }

    .face {
        height: 100%;
        width: 100%;
        backface-visibility: hidden;
    }

    .card-container {
        width: 60px;
        height: 60px;
    }
    .rotate {
        transform: rotateY(180deg);
    }
</style>
