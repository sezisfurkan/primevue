<template>
    <div class="doc-main">
        <div class="doc-intro">
            <h1>ScrollPanel Pass Through</h1>
        </div>
        <DocSections :docs="docs" />
    </div>
    <DocSectionNav :docs="docs" />
</template>

<script>
import DocApiTable from '@/components/doc/DocApiTable.vue';
import { getPTOption } from '@/components/doc/helpers/PTHelper.js';
import PtDoc from './PTDoc.vue';
import PTImage from './PTImage.vue';

export default {
    data() {
        return {
            docs: [
                {
                    id: 'pt.image',
                    label: 'WireFrame',
                    component: PTImage
                },
                {
                    id: 'pt.doc',
                    label: 'ScrollPanel Elements',
                    component: DocApiTable,
                    data: getPTOption('ScrollPanel')
                },
                {
                    id: 'pt.demo',
                    label: 'Example',
                    component: PtDoc
                }
            ]
        };
    }
};
</script>
