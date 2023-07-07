<script lang="ts">
  import { onMount } from 'svelte'
  import { generation, pokemon } from './stores'

  onMount(() => {
    fetch('https://pokeapi.co/api/v2/generation/1/')
      .then((response) => response.json())
      .then((data) => {
        generation.set(data)
      })
      .catch((error) => console.log(error))
  })

  const loaded = new Promise((res, rej) => {
    generation.subscribe((v: any) => {
      pokemon.set([])
      v.pokemon_species &&
        v.pokemon_species.forEach((poke: any) => {
          let id = poke.url.split('/')[6]
          fetch(`https://pokeapi.co/api/v2/pokemon/${id}/`)
            .then((response) => response.json())
            .then((data) => {
              pokemon.update((n: any) => {
                n.push(data)
                return n
              })
            })
            .catch((error) => {
              rej(error)
              console.log(error)
            })
        })
    })
    res(true)
  })

  $: console.log($pokemon)
</script>

<svelte:head>
  <title>JB | Propeller Coding Test</title>
</svelte:head>

<main class="wrap py-8">
  {#await loaded}
    <p>loading...</p>
  {:then hasLoaded}
    {#if hasLoaded}
      <div class="flex gap-8 wrap flex-col">
        <h1 class="text-4xl font-bold">Pokemon</h1>
        {#each $pokemon as indivPokemon}
          <div class="flex gap-8 items-center">
            <img
              class="h-32 w-32"
              src={indivPokemon.sprites.front_default}
              alt={`Sprite of ${indivPokemon.name}`}
            />
            <h1 id={indivPokemon.name} class="text-3xl capitalize">
              {indivPokemon.name}
            </h1>
          </div>
          <div class="flex gap-3 items-center">
            {#each indivPokemon.types as type}
              <p class="capitalize text-lg font-bold">{type.type.name}</p>
            {/each}
          </div>
          <h2 class="text-2xl">Stats</h2>
          <div class="grid grid-cols-2 w-1/3">
            {#each indivPokemon.stats as stat}
              <p class="capitalize">{stat.stat.name}</p>
              <p>{stat.base_stat}</p>
            {/each}
          </div>
        {/each}
      </div>
    {/if}
  {/await}
</main>

<style lang="postcss" global>
  .wrap {
    @apply xl:max-w-[80%] xl:mx-auto mx-6;
  }
</style>
