<script>
  // createEventDispatcher is no longer needed
  import { getContext, onMount, onDestroy } from "svelte";

  // Get SDK and form contexts
  const { styleable } = getContext('sdk');
  const component = getContext('component');
  const formContext = getContext('form');
  const formStepContext = getContext('form-step');

  // Component properties
  export let field;
  export let value = null;
  export let label = "Currency";
  export let disabled = false;
  export let defaultValue = null;
  
  // Budibase will pass the function to trigger the "On change" action into this prop
  export let onChange;

  // Register the field with the Budibase form
  const formApi = formContext?.formApi;
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(field, "number", defaultValue, disabled, null, formStep);

  let fieldState;
  let fieldApi;
  let internalNumber = value;
  let inputElement;
  let id = `currency-input-${Math.random()}`;

  const audFormatter = new Intl.NumberFormat("en-AU", {
    style: "currency",
    currency: "AUD",
  });

  $: unsubscribe = formField?.subscribe((state) => {
    fieldState = state?.fieldState;
    fieldApi = state?.fieldApi;
    const formValue = fieldState?.value;

    if (formValue !== internalNumber && formValue !== undefined) {
      internalNumber = formValue;
      if (document.activeElement !== inputElement) {
        formatAndDisplay(internalNumber);
      }
    }
  });

  onMount(() => {
    setTimeout(() => {
      formatAndDisplay(internalNumber);
    }, 0);
  });

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
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
    if (internalNumber != null && Number.isFinite(internalNumber)) {
      inputElement.value = String(internalNumber);
    } else {
      inputElement.value = "";
    }
  }

  function handleBlur() {
    const currentText = inputElement.value;
    const oldNumber = internalNumber; 
    const newNumber = currentText ? parseFloat(currentText.replace(/[$,]/g, '')) : null;

    if (newNumber != null && Number.isFinite(newNumber)) {
      internalNumber = newNumber;
    } else {
      internalNumber = null;
    }

    if (fieldApi) {
      fieldApi.setValue(internalNumber);
    }

    // THIS IS THE FIX: Call the onChange function directly if it exists and the value changed
    if (oldNumber !== internalNumber && typeof onChange === 'function') {
      // The payload { value: internalNumber } matches the context you defined in the schema
      onChange({
        value: internalNumber
      });
    }

    formatAndDisplay(internalNumber);
  }

</script>

<div class="container" use:styleable={$component.styles}>
  <label for={id} class="spectrum-Body spectrum-Body--sizeS label">
    {label}
  </label>
  <div>
    <input
      class="spectrum-Textfield input"
      style="text-align: right;"
      {disabled}
      {id}
      type="text"
      placeholder="$0.00"
      bind:this={inputElement}
      on:focus={handleFocus}
      on:blur={handleBlur}
    />
  </div>
</div>

<style>
  .container {
    /* Use the --bb-comp-width variable, fallback to 100% */
    width: var(--bb-comp-width, 100%);
  }
  .input {
    /* Use the --bb-comp-height variable */
    height: var(--bb-comp-height, var(--spectrum-global-dimension-size-300));
    /* Make the input fill its container, which is controlled by the width style */
    width: 100%; 
  }
</style>