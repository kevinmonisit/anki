<!--
Copyright: Ankitects Pty Ltd and contributors
License: GNU AGPL, version 3 or later; http://www.gnu.org/licenses/agpl.html
-->
<script lang="ts">
    import { createEventDispatcher } from "svelte";
    import { get } from "svelte/store";

    import ButtonGroup from "../components/ButtonGroup.svelte";
    import IconButton from "../components/IconButton.svelte";
    import Shortcut from "../components/Shortcut.svelte";
    import * as tr from "../lib/ftl";
    import { isApplePlatform } from "../lib/platform";
    import { getPlatformString } from "../lib/shortcuts";
    import { clozeIcon, incrementClozeIcon } from "./icons";
    import { context as noteEditorContext } from "./NoteEditor.svelte";
    import { editingInputIsRichText } from "./rich-text-input";

    const { focusedInput, fields } = noteEditorContext.get();

    // Workaround for Cmd+Option+Shift+C not working on macOS. The keyup approach works
    // on Linux as well, but fails on Windows.
    const event = isApplePlatform() ? "keyup" : "keydown";

    const clozePattern = /\{\{c(\d+)::/gu;
    function getCurrentHighestCloze(increment: boolean): number {
        let highest = 0;

        for (const field of fields) {
            const content = field.editingArea?.content;
            const fieldHTML = content ? get(content) : "";

            const matches: number[] = [];
            let match: RegExpMatchArray | null = null;

            while ((match = clozePattern.exec(fieldHTML))) {
                matches.push(Number(match[1]));
            }

            highest = Math.max(highest, ...matches);
        }

        if (increment) {
            highest++;
        }

        return Math.max(1, highest);
    }

    const dispatch = createEventDispatcher();

    async function onIncrementCloze(): Promise<void> {
        const highestCloze = getCurrentHighestCloze(true);

        dispatch("surround", {
            prefix: `{{c${highestCloze}::`,
            suffix: "}}",
        });
    }

    async function onSameCloze(): Promise<void> {
        const highestCloze = getCurrentHighestCloze(false);

        dispatch("surround", {
            prefix: `{{c${highestCloze}::`,
            suffix: "}}",
        });
    }

    $: disabled = !$focusedInput || !editingInputIsRichText($focusedInput);

    const incrementKeyCombination = "Control+Shift+C";
    const sameKeyCombination = "Control+Alt+Shift+C";
</script>

<ButtonGroup>
    <IconButton
        tooltip="{tr.editingClozeDeletion()} ({getPlatformString(
            incrementKeyCombination,
        )})"
        {disabled}
        on:click={onIncrementCloze}
        --border-left-radius="5px"
    >
        {@html incrementClozeIcon}
    </IconButton>

    <Shortcut
        keyCombination={incrementKeyCombination}
        event="keydown"
        on:action={onIncrementCloze}
    />

    <IconButton
        tooltip="{tr.editingClozeDeletionRepeat()} ({getPlatformString(
            sameKeyCombination,
        )})"
        {disabled}
        on:click={onSameCloze}
        --border-right-radius="5px"
    >
        {@html clozeIcon}
    </IconButton>

    <Shortcut keyCombination={sameKeyCombination} {event} on:action={onSameCloze} />
</ButtonGroup>
