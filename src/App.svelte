<script lang="ts">
  import { onMount } from "svelte";
  import HeartBeatIcon from "./components/HeartBeatIcon.svelte";
  import Quotes from "./data/quotes.json";

  let expectedLifespan = 80;
  let targetDate = localStorage.getItem("targetDate")
    ? new Date(localStorage.getItem("targetDate") as string)
    : null;
  let isDeadLine = localStorage.getItem("isDeadLine")
    ? JSON.parse(localStorage.getItem("isDeadLine")!)
    : undefined;
  let timeRemaining = {};
  let showModal = false;
  let tempTargetDate = targetDate ? targetDate.toISOString().split("T")[0] : "";
  let title = localStorage.getItem("title") ?? "It's ticking, use it wisely";
  let quote = {
    text: "Memento mori—remember death! These are important words. If we kept in mind that we will soon inevitably die, our lives would be completely different",
    author: "Marcus Aurelius",
  };

  function calculateTimeRemaining() {
    const now = new Date();
    let totalMilliseconds;
    if (isDeadLine && targetDate) {
      totalMilliseconds = targetDate.getTime() - now.getTime();
    } else if (targetDate && !isDeadLine) {
      const lifespanDate = new Date(
        targetDate.getFullYear() + expectedLifespan,
        targetDate.getMonth(),
        targetDate.getDate()
      );
      totalMilliseconds = lifespanDate.getTime() - now.getTime();
    } else {
      return {
        years: 0,
        months: 0,
        days: 0,
        hours: 0,
        minutes: 0,
        seconds: 0,
        milliseconds: 0,
      };
    }

    if (totalMilliseconds < 0)
      return {
        years: 0,
        months: 0,
        days: 0,
        hours: 0,
        minutes: 0,
        seconds: 0,
        milliseconds: 0,
      };

    const milliseconds = totalMilliseconds % 1000;
    const totalSeconds = Math.floor(totalMilliseconds / 1000);
    const seconds = totalSeconds % 60;
    const totalMinutes = Math.floor(totalSeconds / 60);
    const minutes = totalMinutes % 60;
    const totalHours = Math.floor(totalMinutes / 60);
    const hours = totalHours % 24;
    const totalDays = Math.floor(totalHours / 24);
    const days = totalDays % 30;
    const totalMonths = Math.floor(totalDays / 30);
    const months = totalMonths % 12;
    const years = Math.floor(totalMonths / 12);

    return { years, months, days, hours, minutes, seconds, milliseconds };
  }

  function fetchDailyQuote() {
    const quotes = Quotes;

    const now = new Date();
    const startOfYear = new Date(now.getFullYear(), 0, 0);
    const dayOfYear = Math.floor(
      (now.getTime() - startOfYear.getTime()) / (1000 * 60 * 60 * 24)
    );

    const quoteIndex = dayOfYear % quotes.length; // Ensures index stays within bounds
    return quotes[quoteIndex];
  }

  function updateCountdown() {
    timeRemaining = calculateTimeRemaining();
  }

  function openModal() {
    showModal = true;
  }
  function closeModal() {
    showModal = false;
  }

  function handleSubmit() {
    title =
      (document.getElementById("title-input") as HTMLInputElement)?.value ||
      title;
    localStorage.setItem("title", title);
    const targetInput = (
      document.getElementById("target-date") as HTMLInputElement
    )?.value;
    if (targetInput) {
      targetDate = new Date(targetInput);
      localStorage.setItem("targetDate", targetDate.toISOString());
    }
    isDeadLine =
      (document.getElementById("isDeadLine") as HTMLInputElement)?.checked ||
      isDeadLine;
    localStorage.setItem("isDeadLine", isDeadLine.toString());

    tempTargetDate = targetDate?.toISOString().split("T")[0] ?? "";
    updateCountdown();
    closeModal();
  }

  onMount(() => {
    updateCountdown();
    quote = fetchDailyQuote();
    const timer = setInterval(updateCountdown, 20);
    return () => clearInterval(timer);
  });
</script>

<main>
  <button
    class="text-md bg-gray-800/90 hover:cursor-pointer hover:opacity-80 rounded-lg w-16 h-8 absolute left-0 top-0 m-2"
    on:click={openModal}>Edit</button
  >
  <div class="mb-6">
    <div id="container" class="relative max-w-6xl">
      <div class="absolute -right-4 top-1/2 transform -translate-y-1/2 z-10">
        <HeartBeatIcon size={50} color="white" />
      </div>
      <div
        id="title"
        class="flex items-center justify-center bg-neutral-950 text-white rounded-2xl px-2"
      >
        <h1
          class="capitalize text-4xl lg:text-6xl whitespace-nowrap overflow-hidden text-ellipsis max-w-full py-1 leading-normal"
        >
          {title}
        </h1>
      </div>
    </div>
    <div class="countdown-container">
      <div class="countdown">
        {#each Object.entries(timeRemaining) as [unit, value]}
          <div class="countdown-item">
            <span class="countdown-value">{value}</span>
            <span class="countdown-label">{unit}</span>
          </div>
        {/each}
      </div>
    </div>
  </div>
  <blockquote class="quote absolute right-0 bottom-10 mr-4">
    <p class="quote-text">"{quote.text}"</p>
    <footer class="quote-author">
      <cite>― {quote.author}</cite>
    </footer>
  </blockquote>

  {#if showModal}
    <div
      class="modal-overlay bg-black/60 fixed top-0 left-0 right-0 bottom-0 flex items-center justify-center"
    >
      <div class="modal bg-neutral-900">
        <h2>Edit Countdown</h2>
        <form on:submit|preventDefault={handleSubmit}>
          <div class="form-group">
            <label for="title-input">Title:</label>
            <input type="text" id="title-input" value={title} required />
          </div>
          <div class="form-group">
            <label for="target-date">Target Date:</label>
            <input
              type="date"
              id="target-date"
              required
              value={tempTargetDate}
            />
          </div>
          <div class="">
            <div class="flex gap-2">
              <label for="isDeadLine">Is DeadLine?</label>
              <input
                type="checkbox"
                bind:checked={isDeadLine}
                value={isDeadLine}
              />
            </div>
            <span class="text-sm text-gray-500">
              (by default it will assume it to be your birth date)
            </span>
          </div>
          <div class="form-actions">
            <button type="submit">Save</button>
            <button type="button" on:click={closeModal}>Cancel</button>
          </div>
        </form>
      </div>
    </div>
  {/if}
</main>

<style>
  @reference "tailwindcss/theme";

  .countdown-container {
    width: 100%;
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
    scrollbar-width: none;
    -ms-overflow-style: none;
  }

  .countdown-container::-webkit-scrollbar {
    display: none;
  }

  .countdown {
    display: inline-flex;
    text-shadow: black 2px 2px 2px;
    justify-content: center;
    gap: 1rem;
    padding: 1rem;
    min-width: min-content;
    margin: 0 auto;
  }

  .countdown-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    flex-shrink: 0;
  }

  .countdown-value {
    font-size: 3rem;
    font-weight: bold;
    min-width: 85px;
  }

  .countdown-label {
    font-size: 1rem;
    text-transform: uppercase;
  }

  @media (max-width: 768px) {
    .countdown {
      justify-content: flex-start;
    }

    .countdown-value {
      font-size: 2.5rem;
      min-width: 70px;
    }
  }

  .quote {
    position: absolute;
    bottom: 1rem;
    right: 2rem;
    max-width: 380px;
    background: rgba(15, 15, 15, 0.541);
    opacity: 0.8;
    backdrop-filter: blur(8px);
    border-left: 2px solid rgba(255, 255, 255, 0.7);
    padding: 1.2rem;
    margin: 0;
    border-radius: 0.5rem;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    transition:
      transform 0.3s ease,
      opacity 0.3s ease;
  }

  .quote:hover {
    transform: translateY(-5px);
    opacity: 1;
  }

  .quote-text {
    color: rgba(255, 255, 255, 0.9);
    font-size: 1rem;
    line-height: 1.6;
    margin: 0;
    font-style: italic;
    text-align: left;
  }

  .quote-author {
    margin-top: 1rem;
    text-align: right;
    text-transform: capitalize;
    color: rgba(255, 255, 255, 0.7);
    font-size: 0.9rem;
  }

  .quote-author cite {
    font-style: normal;
  }

  @media (max-width: 768px) {
    .quote {
      position: relative;
      bottom: auto;
      right: auto;
      margin: 2rem auto;
      max-width: 90%;
    }
  }

  @media (max-width: 480px) {
    .countdown-value {
      font-size: 2rem;
      min-width: 60px;
    }

    .countdown-label {
      font-size: 0.9rem;
    }

    .countdown {
      gap: 0.8rem;
    }
  }

  .modal {
    padding: 20px;
    border-radius: 10px;
    width: 90%;
    max-width: 500px;
  }

  .modal h2 {
    color: white;
    margin-bottom: 20px;
  }

  .form-group {
    margin-bottom: 15px;
  }

  .form-group label {
    display: block;
    color: white;
    margin-bottom: 5px;
  }

  .form-group input {
    width: 100%;
    padding: 8px;
    border: 1px solid #333;
    background-color: #2a2a2a;
    color: white;
    border-radius: 4px;
  }

  .form-actions {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
    margin-top: 20px;
  }

  .form-actions button {
    padding: 8px 15px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  .form-actions button[type="submit"] {
    background-color: #4caf50;
    color: white;
  }

  .form-actions button[type="submit"]:hover {
    background-color: #45a049;
  }
</style>
