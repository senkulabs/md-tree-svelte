<script>
import { generateTree } from "./lib/generate-tree";
import { parseInput } from "./lib/parse-input";

const initialValue = `Edit me to generate
  a
    nice
      tree
        diagram!
        :)
  Use indentation
    to indicate
      file
      and
      folder
      nesting.
    - You can even
      - use
        - markdown
        - bullets!`.trim();

  let value = $state(initialValue);
  let copied = $state(false);
  let shared = $state(false);
  let fancyMode = $state(true);
  let showTrailingSlashDir = $state(true);
  let showFullPath = $state(false);
  let showRootDot = $state(true);

  let textarea;

  $effect(() => {
    const url = new URL(window.location.href);
    const searchParams = url.searchParams;
    
    if (searchParams.has('snippet')) {
      const result = JSON.parse(atob(searchParams.get('snippet')));
      if (result.hasOwnProperty('content')) {
        value = JSON.parse(result.content);
      }
      if (result.hasOwnProperty('charset')) {
        fancyMode = result.charset === 'utf-8' ? true : false;
      }
      if (result.hasOwnProperty('trailingSlashDir')) {
        showTrailingSlashDir = result.trailingSlashDir;
      }
      if (result.hasOwnProperty('fullPath')) {
        showFullPath = result.fullPath;
      }
      if (result.hasOwnProperty('rootDot')) {
        showRootDot = result.rootDot;
      }
    }
  });

  $effect(() => {
    textarea.focus();
    // Move cursor to the end of text
    const length = textarea.value.length;
    textarea.setSelectionRange(length, length);
  });

  const options = $derived({
    charset: fancyMode ? 'utf-8' : 'ascii',
    trailingSlashDir: showTrailingSlashDir,
    fullPath: showFullPath,
    rootDot: showRootDot,
  });

  function handleTabKey(event) {
    if (event.key === 'Tab') {
      event.preventDefault();

      const { selectionStart, selectionEnd } = event.target;
      const value = event.target.value;
      
      // Insert tab at cursor position
      event.target.value = value.substring(0, selectionStart) + `\t` + value.substring(selectionEnd);

      // Move cursor after tab
      event.target.setSelectionRange(
        selectionStart + 1,
        selectionStart + 1
      )
    }
  }

  async function handleCopy() {
    try {
      await navigator.clipboard.writeText(generateTree(parseInput(value), options));
      copied = true;

      // Reset the "copied" message after 2 seconds
      setTimeout(() => {
        copied = false;
      }, 2000);
    } catch (error) {
      console.error('Failed to copy text:', error);
    }
  }

  async function handleShare() {
    try {
      let data = {
        content: JSON.stringify(value),
        ...options
      };
      
      let url = new URL(window.location.origin);
      url.searchParams.append('snippet', btoa(JSON.stringify(data)));
      if (navigator.canShare()) {
        await navigator.share(url.href);
      } else {
        await navigator.clipboard.writeText(url.href);
      }
      
      shared = true;
  
      setTimeout(() => {
        shared = false;
      }, 2000);
    } catch (error) {
      console.error('Failed to shared', error);
    }
  }

  function handleFancyMode() {
    fancyMode = !fancyMode;
  }

  function handleTrailingSlashDir() {
    showTrailingSlashDir = !showTrailingSlashDir;
  }

  function handleFullPath() {
    showFullPath = !showFullPath;
  }

  function handleRootDot() {
    showRootDot = !showRootDot;
  }
</script>

<div class="container p-2 mx-auto">
  <h1 style="text-align: center;">Markdown Tree Generator</h1>
  <p style="margin: .5rem;">Generate tree for markdown use case. The idea of this project comes from <a href="https://tree.nathanfriend.com" target="_blank">tree.nathanfriend.com</a> but with Svelte 5.</p>
  <div class="editor">
    <textarea bind:this={textarea} style="width: 480px; height: 320px; tab-size: 2; font-size: 1rem;" bind:value={value} onkeydown={handleTabKey}></textarea>
    <div class="tree">
      { generateTree(parseInput(value), options) }
    </div>
  </div>
  <div class="control">
    <button onclick={handleCopy}>{ copied ? 'Copied' : 'Copy' }</button>
    <button onclick={handleShare}>{ shared ? 'URL copied' : 'Share' }</button>
    <label for="fancy-mode">
      <input type="checkbox" id="fancy-mode" onchange={handleFancyMode} checked={fancyMode} />Fancy
    </label>
    <label for="trailing-mode">
      <input type="checkbox" id="trailing-mode" onchange={handleTrailingSlashDir} checked={showTrailingSlashDir} />Trailing /
    </label>
    <label for="fullpath-mode">
      <input type="checkbox" id="fullpath-mode" onchange={handleFullPath} checked={showFullPath} />Full Path
    </label>
    <label for="root-dot-mode">
      <input type="checkbox" id="root-dot-mode" onchange={handleRootDot} checked={showRootDot} />Root .
    </label>
  </div>
</div>