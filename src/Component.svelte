<script>
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
  export let onChange;
  
  // 1. Add the new currency prop from the schema settings
  export let currency = "AUD";

  const formApi = formContext?.formApi;
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(field, "number", defaultValue, disabled, null, formStep);

  let fieldState;
  let fieldApi;
  let internalNumber = value;
  let inputElement;
  let id = `currency-input-${Math.random()}`;

  // 2. Make the formatter dynamic and reactive
  let currencyFormatter;
  let placeholder = "0.00"; // Default placeholder
  
  // Helper function to get a suitable locale for a currency
function getLocaleForCurrency(currencyCode) {
    switch (currencyCode) {
      // Americas
      case 'USD': return 'en-US'; // US Dollar
      case 'CAD': return 'en-CA'; // Canadian Dollar
      case 'MXN': return 'es-MX'; // Mexican Peso
      case 'BRL': return 'pt-BR'; // Brazilian Real

      // Europe
      case 'EUR': return 'de-DE'; // Euro (German locale is a common choice)
      case 'GBP': return 'en-GB'; // British Pound
      case 'CHF': return 'de-CH'; // Swiss Franc
      case 'SEK': return 'sv-SE'; // Swedish Krona
      case 'NOK': return 'nb-NO'; // Norwegian Krone
      case 'RUB': return 'ru-RU'; // Russian Ruble
      case 'TRY': return 'tr-TR'; // Turkish Lira

      // Asia & Oceania
      case 'JPY': return 'ja-JP'; // Japanese Yen
      case 'CNY': return 'zh-CN'; // Chinese Yuan Renminbi
      case 'INR': return 'en-IN'; // Indian Rupee
      case 'KRW': return 'ko-KR'; // South Korean Won
      case 'SGD': return 'en-SG'; // Singapore Dollar
      case 'HKD': return 'zh-HK'; // Hong Kong Dollar
      case 'AUD': return 'en-AU'; // Australian Dollar
      case 'NZD': return 'en-NZ'; // New Zealand Dollar

      // Africa
      case 'ZAR': return 'en-ZA'; // South African Rand
      
      default: return 'en-US'; // A safe fallback
    }
}

  // This is a Svelte reactive block. It will re-run whenever `currency` changes.
  $: {
    const locale = getLocaleForCurrency(currency);
    currencyFormatter = new Intl.NumberFormat(locale, {
      style: "currency",
      currency: currency,
    });
    // Update the placeholder dynamically
    placeholder = currencyFormatter.format(0);
    // Re-format the current value if the currency is changed in the editor
    formatAndDisplay(internalNumber);
  }

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
    if (inputElement && currencyFormatter) { // Ensure formatter is created
      if (number != null && Number.isFinite(number)) {
        inputElement.value = currencyFormatter.format(number); // 3. Use the dynamic formatter
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
    if (oldNumber !== internalNumber && typeof onChange === 'function') {
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
      {placeholder}
      bind:this={inputElement}
      on:focus={handleFocus}
      on:blur={handleBlur}
    />
  </div>
</div>

<style>
  .container {
    width: var(--bb-comp-width, 100%);
  }
  .input {
    height: var(--bb-comp-height, var(--spectrum-global-dimension-size-300));
    width: 100%; 
  }
</style>