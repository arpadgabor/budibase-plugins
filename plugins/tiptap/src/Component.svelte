<script lang="ts">
  import { getContext, onDestroy, type ComponentEvents } from 'svelte'
  import Editor from './Editor.svelte'

  const component = getContext<{ styles: any; subscribe: () => void }>(
    'component'
  )
  const formContext = getContext<{ formApi: any }>('form')
  const formStepContext = getContext<any>('form-step')
  const sdk = getContext<{ styleable: any }>('sdk')

  export let field
  export let label
  // export let defaultValue

  const formApi = formContext?.formApi
  $: formStep = formStepContext ? $formStepContext || 1 : 1
  $: formField = formApi?.registerField(
    field,
    'json',
    {},
    false,
    null,
    formStep
  )

  let fieldApi: { setValue: (value: any) => void; deregister: () => void }
  let fieldState

  $: unsubscribe = formField?.subscribe(value => {
    fieldState = value?.fieldState
    fieldApi = value?.fieldApi
    console.log(fieldState)
  })

  onDestroy(() => {
    fieldApi?.deregister()
    unsubscribe?.()
  })

  // data: JSON object from tiptap
  function onUpdate(data: ComponentEvents<Editor>['update']) {
    console.log('Updating', data)
    fieldApi?.setValue(data.detail)
  }

  const fieldGroupContext = getContext<any>('field-group')
  const labelPos = fieldGroupContext?.labelPosition || 'above'
  $: labelClass = labelPos === 'above' ? '' : `spectrum-FieldLabel--${labelPos}`
</script>

<div use:sdk.styleable={$component.styles}>
  <div class="spectrum-Form-item">
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || ' '}
    </label>
  </div>

  {#if fieldState}
    <div
      class="spectrum-Textfield spectrum-Textfield-input"
      style="height: auto !important;"
    >
      <Editor
        content={fieldState.value || '<p>Type something </p>'}
        editable={!!formContext}
        on:update={onUpdate}
      />
    </div>
  {/if}
</div>
