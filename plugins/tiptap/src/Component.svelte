<script lang="ts">
  import { getContext, onDestroy, onMount } from 'svelte'
  import { Editor } from '@tiptap/core'
  import StarterKit from '@tiptap/starter-kit'
  import {} from '@budibase/bb-ui'

  const component = getContext<{ styles: any; subscribe: () => void }>(
    'component'
  )
  const formContext = getContext<{ formApi: any }>('form')
  const formStepContext = getContext<any>('form-step')
  const sdk = getContext<{ styleable: any }>('sdk')

  export let field
  export let label
  export let defaultValue

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
    console.log(fieldState)
    fieldState = value?.fieldState
    fieldApi = value?.fieldApi
  })

  let element: HTMLDivElement
  let editor: Editor

  onMount(() => {
    console.log(fieldState)
    editor = new Editor({
      element: element,
      editable: !!formContext,
      extensions: [StarterKit],
      content: defaultValue
        ? JSON.parse(defaultValue)
        : fieldState?.value || '<p>Type something </p>',
      onTransaction: () => {
        // force re-render so `editor.isActive` works as expected
        editor = editor
      },
    })

    editor.on('update', e => fieldApi?.setValue(e.editor.getJSON()))
  })

  onDestroy(() => {
    fieldApi?.deregister()
    unsubscribe?.()
    if (editor) {
      editor.destroy()
    }
  })
</script>

<div use:sdk.styleable={$component.styles}>
  {#if editor && editor.isEditable}
    <button
      on:click={() => editor.chain().focus().toggleHeading({ level: 1 }).run()}
      class:active={editor.isActive('heading', { level: 1 })}
    >
      H1
    </button>
    <button
      on:click={() => editor.chain().focus().toggleHeading({ level: 2 }).run()}
      class:active={editor.isActive('heading', { level: 2 })}
    >
      H2
    </button>
    <button
      on:click={() => editor.chain().focus().setParagraph().run()}
      class:active={editor.isActive('paragraph')}
    >
      P
    </button>
  {/if}

  <div bind:this={element} />
</div>
