<script lang="ts">
  import { onDestroy, onMount, createEventDispatcher } from 'svelte'

  import { Editor, type Content } from '@tiptap/core'
  import StarterKit from '@tiptap/starter-kit'
  import BubbleMenu from '@tiptap/extension-bubble-menu'
  import FloatingMenu from '@tiptap/extension-floating-menu'

  const emit = createEventDispatcher()

  export let content: Content
  export let editable: boolean = true

  let editor: Editor
  let element: HTMLDivElement
  let floatingMenu: HTMLDivElement
  let bubbleMenu: HTMLDivElement

  onMount(() => {
    editor = new Editor({
      editable,
      element,
      content,
      extensions: [
        StarterKit,
        BubbleMenu.configure({
          element: bubbleMenu,
          tippyOptions: {
            duration: 100,
          },
        }),
        FloatingMenu.configure({
          element: floatingMenu,
          tippyOptions: {
            duration: 100,
          },
        }),
      ],

      // force re-render so `editor.isActive` works as expected
      onTransaction() {
        editor = editor
      },
      // onUpdate({ editor }) {
      //   emit('update', editor.getJSON())
      // },
    })

    editor.on('update', e => emit('update', e.editor.getJSON()))
  })

  onDestroy(() => {
    editor && editor.destroy()
  })
</script>

<div style="width: 100%">
  <div bind:this={floatingMenu}>
    <button
      on:click={() => editor?.chain().focus().toggleHeading({ level: 1 }).run()}
      class:active={editor?.isActive('heading', { level: 1 })}
    >
      H1
    </button>
    <button
      on:click={() => editor?.chain().focus().toggleHeading({ level: 2 }).run()}
      class:active={editor?.isActive('heading', { level: 2 })}
    >
      H2
    </button>
    <button
      on:click={() => editor?.chain().focus().setParagraph().run()}
      class:active={editor?.isActive('paragraph')}
    >
      P
    </button>
  </div>

  <div bind:this={bubbleMenu} class="bubble-menu">
    <button
      on:click={() => editor?.chain().focus().toggleBold().run()}
      class:active={editor?.isActive('bold')}
    >
      B
    </button>
    <button
      on:click={() => editor?.chain().focus().toggleItalic().run()}
      class:active={editor?.isActive('italic')}
    >
      I
    </button>
  </div>

  <div bind:this={element} />
</div>
