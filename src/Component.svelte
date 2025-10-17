<script>
  import { getContext, createEventDispatcher, onMount } from "svelte";
  // import { NumberInput, css } from '@svelteuidev/core';
  const dispatch = createEventDispatcher();

  const { styleable } = getContext("sdk");
  const component = getContext("component");

  // export let text = '';
  export let value = null;
  export let label = "Currency";

  let internalNumber = value;
  let inputElement;
  let id = Math.floor(Math.random()*100);

  const audFormatter = new Intl.NumberFormat("en-AU", {
    style: "currency",
    currency: "AUD",
  });

  onMount(() => {
    setTimeout(() => {
      formatAndDisplay(internalNumber);
    }, 0);
  });

  function formatAndDisplay(number) {
    if (inputElement) {
      if (number != null && Number.isFinite(number)) {
        inputElement.value = audFormatter.format(number);
      } else {
        inputElement.value = "";
      }
    }
  }

  function handleFocus() {
    if (internalNumber != null) {
      inputElement.value = String(internalNumber);
    } else {
      inputElement.value = "";
    }
  }

  function handleBlur() {
    const currentText = inputElement.value;
    const newNumber = currentText ? parseFloat(currentText.replace(/,/g, '')) : null;

    if (Number.isFinite(newNumber)) {
      internalNumber = newNumber;
    } else {
      internalNumber = null;
    }

    dispatch("setValue", internalNumber);
    formatAndDisplay(internalNumber);
  }

  // const styles = css({
  //     [`.svelteui-Input-input`]: {
  //       backgroundColor: "var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50))",
  //       borderColor: "var(--spectrum-textfield-m-border-color, var(--spectrum-alias-border-color))",
  //       color: "var(--spectrum-textfield-m-text-color, var(--spectrum-alias-text-color))"
  //     },
  //     [`.svelteui-NumberInput-control`]: {
  //       borderColor: "var(--spectrum-textfield-m-border-color, var(--spectrum-alias-border-color))"
  //     },
  //     [`.svelteui-NumberInput-control:hover`]: {
  //       backgroundColor: "var(--spectrum-button-primary-m-background-color-hover, var(--spectrum-global-color-gray-800)) !important",
  //       borderColor: "var(--spectrum-button-primary-m-text-color-hover, var(--spectrum-global-color-gray-50)) !important"
  //     }
  // })
</script>

<div use:styleable={$component.styles}>
  <label for={id} class="spectrum-Body spectrum-Body--sizeS label">
    {label}
  </label>
  <div >
  <input
    class="spectrum-Textfield input"
    style="text-align: right;"
    {id}
    type="text"
    placeholder="$0.00"
    bind:this={inputElement}
    on:focus={handleFocus}
    on:blur={handleBlur}
  />
  </div>
</div>
