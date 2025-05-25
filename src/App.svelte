<script lang="ts">
  import * as Plot from "@observablehq/plot";
  import { onMount } from "svelte";

  let isDark = $state(false);

  onMount(() => {
    const media = window.matchMedia("(prefers-color-scheme: dark)");
    isDark = media.matches;

    // react to changes in system setting
    media.addEventListener("change", (e) => {
      isDark = e.matches;
    });
  });

  let current = $state(0);

  const parties = [
    {
      party: "Conservative",
      leftRight: 4.44,
      authLib: 7.34,
      vote: 23.7,
      color: "#0087dc",
    },
    {
      party: "Labour",
      leftRight: 2.3,
      authLib: 5.4,
      vote: 33.7,
      color: "#e4003b",
    },
    {
      party: "Lib Dem",
      leftRight: 2.78,
      authLib: 5.34,
      vote: 12.1,
      color: "#faa619",
    },
    {
      party: "SNP",
      leftRight: 1.8,
      authLib: 4.95,
      vote: 2.5,
      color: "#fdf38e",
    },
    {
      party: "Green",
      leftRight: 1.79,
      authLib: 4.38,
      vote: 6.4,
      color: "#06a95b",
    },
    {
      party: "Reform",
      leftRight: 3.22,
      authLib: 7.63,
      vote: 14.3,
      color: "#12b6cf",
    },
  ];

  function closestParty() {
    return parties
      .map((p) => {
        return {
          party: p.party,
          dist:
            Math.pow(p.leftRight - score[0], 2) +
            Math.pow(p.authLib - score[1], 2),
        };
      })
      .sort((a, b) => a.dist - b.dist)[0];
  }

  const questions = [
    "Government should redistribute income from the better off to those who are less well off",
    "Big business takes advantage of ordinary people",
    "Ordinary working people do not get their fair share of the nation's wealth",
    "There is one law for the rich and one for the poor",
    "Management will always try to get the better of employees if it gets the chance",
    "Young people today don't have enough respect for traditional British values",
    "For some crimes, the death penalty is the most appropriate sentence",
    "Schools should teach children to obey authority",
    "Censorship of films and magazines is necessary to uphold moral standards",
    "People who break the law should be given stiffer sentences",
  ];

  let answers = $state(Array(questions.length).fill(0));

  const clicked = (n: number) => {
    answers[current] = n;
    if (current < questions.length) current++;
  };

  let score = $derived.by(() => {
    let lr = 0;
    let al = 0;

    for (let i = 0; i < 5; i++) {
      lr += answers[i];
    }

    for (let i = 5; i < 10; i++) {
      al += answers[i];
    }

    return [10 - lr / 2, al / 2];
  });

  let plotDiv = $state<HTMLDivElement | null>(null);
  $effect(() => {
    if (current < questions.length || !plotDiv) return;

    // Clean the container
    plotDiv.replaceChildren();

    const userPoint = {
      party: "You",
      leftRight: score[0],
      authLib: score[1],
      isUser: true,
    };

    const plot = Plot.plot({
      marginLeft: 40,
      marginBottom: 40,
      width: 500,
      height: 500,
      x: {
        label: "Left (0) → Right (10)",
        domain: [0, 10],
        ticks: 11,
      },
      y: {
        label: "Liberal (0) → Authoritarian (10)",
        domain: [0, 10],
        ticks: 11,
      },
      r: {
        domain: [0, Math.max(...parties.map((d) => d.vote))],
        range: [4, 20], // adjust min/max dot size
      },
      marks: [
        Plot.dot(parties, {
          x: "leftRight",
          y: "authLib",
          stroke: "color",
          fill: "color",
          title: (d) => d.party,
          r: "vote",
          tip: true,
        }),
        Plot.dot([userPoint], {
          x: "leftRight",
          y: "authLib",
          fill: isDark ? "white" : "black",
          stroke: "black",
          title: "You",
          r: 6,
        }),
        Plot.text([userPoint], {
          x: "leftRight",
          y: "authLib",
          text: "party",
          dy: -12,
          fontSize: 12,
        }),
        Plot.ruleX([5], {
          stroke: isDark ? "white" : "black",
          strokeOpacity: 0.5,
        }),
        Plot.ruleY([5], {
          stroke: isDark ? "white" : "black",
          strokeOpacity: 0.5,
        }),
      ],
    });

    plotDiv.append(plot);
  });
</script>

{#if current < questions.length}
  <div>
    <h2>Question {current + 1}</h2>
    <p>{questions[current]}</p>
  </div>

  <div id="answers">
    {#each ["Strongly disagree", "disagree", "Neither agree nor disagree", "Agree", "Strongly agree"] as answer, i}
      <button aria-label={answer} onclick={() => clicked(i)}>{answer}</button>
    {/each}
  </div>

  <div>
    <button
      aria-label="Go back"
      onclick={() => {
        current--;
      }}
      disabled={current === 0}>Go back</button
    >
  </div>
{:else}
  <p>
    Your position: Left–Right = {score[0].toFixed(2)}, Auth–Lib = {score[1].toFixed(
      2,
    )}
  </p>
  <p>Your closest party: {closestParty().party}</p>
  <div bind:this={plotDiv}></div>
  <button
    aria-label="Reset"
    onclick={() => {
      current = 0;
      answers.fill(0);
    }}>Reset</button
  >
{/if}

<style>
  #answers {
    display: flex;
    flex-direction: column;
    align-items: center; /* aligns buttons to the left edge */
    gap: 0.5rem;
    padding: 0.5rem;
  }

  #answers button {
    width: 14rem; /* adjust this value to fit your longest label */
  }
</style>
