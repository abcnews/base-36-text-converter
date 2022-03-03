<script context="module" lang="ts">
  type HTMLElementEvent<T extends HTMLElement> = Event & {
    target: T;
  };
</script>

<script lang="ts">
  import { decode, encode } from '@abcnews/base-36-text';
  import { onMount } from 'svelte';

  let textEl: HTMLTextAreaElement;
  let base36El: HTMLTextAreaElement;
  let onChangeText: (event: Event) => void;
  let onChangeBase36: (event: Event) => void;
  let onCopyText: (event: Event) => Promise<void>;
  let onCopyBase36: (event: Event) => Promise<void>;
  let errorMessage: string;

  function createConversionHandler(
    converter: (value: string) => string,
    outputEl: HTMLTextAreaElement
  ) {
    function asynchronouslyHandleEvent(event: HTMLElementEvent<HTMLTextAreaElement>) {
      errorMessage = null;

      const inputValue = event.target.value;

      try {
        outputEl.value = inputValue ? converter(inputValue) : '';
      } catch (err) {
        outputEl.value = '';
        console.error(err);

        if (err.name === 'SyntaxError') {
          errorMessage = 'Base36 cannot be decoded';
        } else {
          errorMessage = err.message;
        }
      }
    }

    let timer: NodeJS.Timeout;

    return (event: Event) => {
      clearTimeout(timer);
      timer = setTimeout(asynchronouslyHandleEvent, 500, event);
    };
  }

  function createCopyHandler(sourceEl: HTMLTextAreaElement) {
    return () => navigator.clipboard.writeText(sourceEl.value);
  }

  onMount(() => {
    onChangeText = createConversionHandler(encode, base36El);
    onChangeBase36 = createConversionHandler(decode, textEl);
    onCopyText = createCopyHandler(textEl);
    onCopyBase36 = createCopyHandler(base36El);
  });
</script>

<div>
  <h1>Base36 ↔️ Text</h1>
  <label>
    <span>Text</span>
    <button on:click={onCopyText}>Copy</button>
  </label>
  <textarea bind:this={textEl} on:input={onChangeText} />
  <label>
    <span>Base36</span>
    <button on:click={onCopyBase36}>Copy</button>
  </label>
  <textarea bind:this={base36El} on:input={onChangeBase36} />
  <p>{errorMessage || ''}</p>
</div>

<style lang="scss">
  div {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 1rem;
    width: 100%;
    height: 100%;
    min-height: 100vh;
    background-color: #ffc100;
  }

  h1 {
    margin: 0;
    text-align: center;
  }

  div > * {
    display: block;
    margin: 0;
    width: 100%;
    max-width: 24rem;
  }

  label {
    margin-top: 2rem;
    padding: 0 0 0 0.125rem;
    font-size: 1.125rem;
    line-height: 1.5;
  }

  span {
    font-weight: bold;
  }

  button {
    float: right;
    margin: 0.125rem 0 0 0;
    border: 0;
    padding: 0 0.5rem;
    background-color: #000;
    color: #fff;
    font-size: 0.75rem;
    line-height: 1.75;
    letter-spacing: 0.05em;
    text-transform: uppercase;
    cursor: pointer;
  }

  textarea {
    border: 0;
    padding: 0.5rem;
    min-height: 16rem;
    background-color: #fff;
    font-size: 1rem;
    font-family: monospace;
    tab-size: 2;
    resize: vertical;
  }

  textarea ~ textarea {
    min-height: 8rem;
  }

  p {
    padding: 0.125rem;
    min-height: 3rem;
    color: red;
    line-height: 1.5;
  }
</style>
