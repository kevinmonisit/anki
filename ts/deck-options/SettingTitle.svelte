<!--
    Copyright: Ankitects Pty Ltd and contributors
    License: GNU AGPL, version 3 or later; http://www.gnu.org/licenses/agpl.html
-->
<div class="setting-title" on:click>
    <slot />
</div>

<style lang="scss">
    .setting-title {
        cursor: help;
        &:hover {
            text-decoration: underline;
            text-decoration-style: dashed;
        }
    }
</style>
