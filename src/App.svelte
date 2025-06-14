<script lang="ts">
  import * as Plot from "@observablehq/plot";
  import PlotComponent from "./lib/Plot.svelte";
  import { onMount } from "svelte";
  import { parties, qualifications, age, tenure } from "./data";

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

  let selected = $state("parties");
  const options = ["parties", "qualifications", "age", "tenure"];

  let context = $derived.by(() => {
    switch (selected) {
      case "parties":
        return parties;
      case "qualifications":
        return qualifications;
      case "age":
        return age;
      case "tenure":
        return tenure;
      default:
        return [];
    }
  });

  function closestGroup() {
    return context
      .map((p) => {
        return {
          label: p.label,
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

  let userPoint = $derived({
    label: "You",
    leftRight: score[0],
    authLib: score[1],
    isUser: true,
  });

  let plotOptions = $derived({
    color: {
      legend: true,
      type: "categorical",
      domain: context.map((d) => d.label),
      range: selected === "parties" ? parties.map((d) => d.color) : undefined,
    },
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
      domain: [0, Math.max(...parties.map((d) => d.size))],
      range: [4, 20], // adjust min/max dot size
    },
    marks: [
      Plot.dot(context, {
        x: "leftRight",
        y: "authLib",
        stroke: "label",
        fill: "label",
        title: "label",
        r: "size",
      }),
      Plot.dot([userPoint], {
        x: "leftRight",
        y: "authLib",
        fill: isDark ? "white" : "black",
        stroke: "black",
        title: "You",
        r: 6,
      }),
      Plot.tip(
        context,
        Plot.pointer({
          x: "leftRight",
          y: "authLib",
          title: "label",
          fill: isDark ? "black" : "white",
          fontSize: 12,
        }),
      ),
      Plot.text([userPoint], {
        x: "leftRight",
        y: "authLib",
        text: "label",
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
  <p>
    Your closest {selected === "parties" ? "party" : "group"}: {closestGroup()
      .label}
  </p>
  <p>
    Context: <select bind:value={selected}>
      {#each options as option}
        <option value={option}>
          {option}
        </option>
      {/each}
    </select>
  </p>

  <PlotComponent options={plotOptions} />
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
