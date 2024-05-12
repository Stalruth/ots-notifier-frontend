<script>
import 'mvp.css';
import '$lib/css/stalruth.css';
import '$lib/css/app.css';

let webhook = '';

let result = '';
let resultDetail = '';

function getWebhookField(url, field) {
  try {
    return (new URL(webhook)).pathname.split('/')[field];
  } catch {
    return '';
  }
}

$: webhookId = getWebhookField(webhook, 3);
$: webhookToken = getWebhookField(webhook, 4);

async function submitWebhook(e) {
  if (webhookId.length === 0 || webhookId.length > 20) {
    result = 'Error';
    resultDetail = 'Not a valid Webhook URL';
    return;
  }
  if (webhookToken.length === 0 || webhookToken.length > 150) {
    result = 'Error';
    resultDetail = 'Not a valid Webhook URL';
    return;
  }
  const response = await fetch('https://api.standings.stalruth.dev/api/create', {
    'method': 'POST',
    'body': new URLSearchParams({'snowflake': webhookId, 'token': webhookToken})
  });
  if (response.ok) {
    result = 'Success';
    const body = await response.json();
    resultDetail = 'Your webhook has been updated! You will now receive notifications when Open Teamlists from a major event are published.';
    if (body.result === 'created') {
      resultDetail = 'Your webhook has been added! You will now receive notifications when Open Teamlists from a major event are published.';
    }
  } else {
    result = 'Error';
    const body = await response.json();
    if (body.result === 'failed validation') {
      resultDetail = 'Not a valid webhook URL';
    } else if (body.result === 'conflict') {
      resultDetail = 'Webhook is already in the database - if it isn\'t working please try again with a new webhook.'
    } else {
      resultDetail = 'Something went wrong, but it\'s probably not your fault.';
    }
  }
}
</script>

<svelte:head>
  <title>Open Teamsheet Notifier</title>
  <meta property="og:title" content="Open Teamsheet Notifier" />
  <meta property="og:url" content="https://ots-notifier.stalruth.dev/" />
  <meta property="og:description" content="Receive a Discord message when the Open Teamlists from a major event are published." />
  <meta name="description" content="Receive a Discord message when the Open Teamlists from a major event are published." />
  <link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
  <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
  <link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
  <link rel="manifest" href="/site.webmanifest">
</svelte:head>

<main>
  <h1>Open Teamsheet Notifier</h1>

  <p>Get notified in your Discord server when Open Teamlists are published in 2 simple steps!</p>

  <h2>1. Make a Discord Webhook</h2>

  <p>If you don't know how to do this, there's an <a href="https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks">official guide here!</a></p>

  <h2>2. Enter your Discord Webhook URL below</h2>

  <form on:submit|preventDefault={submitWebhook}>
    <label>
      URL <input type="text" name="webhook" bind:value={webhook}>
    </label>
    <input type="submit" value="Submit">
  </form>

  {#if result !== ''}
    <div class={result.toLowerCase()}>
      <p>{result}</p>
      <p>{resultDetail}</p>
    </div>
  {/if}
</main>
