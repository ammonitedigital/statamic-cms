<template>

    <div class="flex">
        <slot name="branch-action" :branch="page">
            <div v-if="editable" class="page-move w-6" />
        </slot>
        <div class="flex items-center flex-1 p-2 rtl:mr-2 ltr:ml-2 text-xs leading-normal">
            <div class="flex items-center grow" @click="$emit('branch-clicked', page)">
                <div class="little-dot rtl:ml-2 ltr:mr-2" :class="getStatusClass()" v-tooltip="getStatusTooltip()" />
                <svg-icon name="home-page" class="rtl:ml-2 ltr:mr-2 h-4 w-4 text-gray-800" v-if="isRoot" v-tooltip="__('This is the root page')" />
                <a
                    @click.prevent="$emit('edit', $event)"
                    :class="{ 'text-sm font-medium': isTopLevel }"
                    :href="page.edit_url"
                    v-text="title" />

                <span v-if="showSlugs" class="rtl:mr-2 ltr:ml-2 font-mono text-gray-700 text-2xs pt-px">
                    {{ isRoot ? '/' : page.slug }}
                </span>

                <button
                    v-if="hasChildren"
                    class="p-2 text-gray-600 hover:text-gray-700 transition duration-100 outline-none flex"
                    :class="{ '-rotate-90': !isOpen }"
                    @click="$emit('toggle-open')"
                >
                    <svg-icon name="micro/chevron-down-xs" class="h-1.5" />
                </button>

                <div v-if="page.collection && editable" class="rtl:mr-4 ltr:ml-4 flex items-center">
                    <svg-icon name="light/content-writing" class="w-4 h-4" />
                    <div class="rtl:mr-1 ltr:ml-1">
                        <a :href="page.collection.create_url" v-text="__('Add')" />
                        <span class="text-gray">/</span>
                        <a :href="page.collection.edit_url" v-text="__('Edit')" />
                    </div>
                </div>
            </div>

            <div class="rtl:pl-2 ltr:pr-2 flex items-center">
                <div v-if="showBlueprint && page.entry_blueprint" v-text="__(page.entry_blueprint.title)" class="shrink text-4xs text-gray-600 uppercase ml-4" />

                <slot name="branch-icon" :branch="page" />

                <dropdown-list class="rtl:mr-4 ltr:ml-4" :class="{'invisible': isRoot, 'hidden': !editable}">
                    <slot name="branch-options"
                        :branch="page"
                        :depth="depth"
                        :remove-branch="remove"
                        :orphan-children="orphanChildren"
                    />
                </dropdown-list>
            </div>

        </div>
    </div>

</template>

<script>
import * as th from 'tree-helper';

export default {

    props: {
        page: Object,
        depth: Number,
        root: Boolean,
        vm: Object,
        firstPageIsRoot: Boolean,
        isOpen: Boolean,
        hasChildren: Boolean,
        showSlugs: Boolean,
        showBlueprint: Boolean,
        editable: Boolean,
    },

    data() {
        return {
            editing: false,
        }
    },

    computed: {

        isTopLevel() {
            return this.depth === 1;
        },

        isRoot() {
            if (!this.firstPageIsRoot) return false;
            if (!this.isTopLevel) return false;

            const firstNodeId = this.vm.data.parent.children[0].id;
            return this.page.id === firstNodeId;
        },

        isEntry() {
            return Boolean(this.page.id);
        },

        isLink() {
            return !this.page.id && this.page.title && this.page.url;
        },

        isText() {
            return this.page.title && !this.page.url;
        },

        title() {
            return this.page.title || this.page.entry_title || this.page.url;
        }

    },

    methods: {

        getStatusClass() {
            switch (this.page.status) {
                case 'published':
                    return 'bg-green-600';
                case 'draft':
                    return 'bg-gray-400';
                default:
                    return 'bg-transparent border border-gray-600';
            }
        },

        getStatusTooltip() {
            let label = __(this.page.status) || __('Text item');

            return label[0].toUpperCase() + label.slice(1);
        },

        remove() {
            const store = this.page._vm.store;
            store.deleteNode(this.page);
            this.$emit('removed', store);
        },

        orphanChildren() {
            const store = this.page._vm.store;

            this.vm.data.children.slice().forEach((child) =>
                th.insertBefore(child, this.vm.data)
            );

            this.$emit('children-orphaned', store);
        }

    }

}
</script>
