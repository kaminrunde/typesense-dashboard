<template>
  <div v-if="currentCollection" class="column no-wrap search-result-item">
    <q-list dense>
      <q-item v-for="field in currentCollection.fields" :key="field.name">
        <q-item-section side class="q-mt-sm text-body2">
          <q-item-label caption>
            {{ field.name }}
            <span v-if="item && Array.isArray(item[field.name])"
              >[{{ item[field.name].length }}]</span
            ></q-item-label
          >
          <q-item-label
            class="overflow-hidden text-no-wrap text-ellipsis"
            :title="item[field.name]"
          >
            <div
              v-if="item && Array.isArray(item._highlightResult[field.name])"
              class="array-field"
            >
              <div
                v-for="(result, index) in item._highlightResult[field.name]"
                v-html="result.value"
                :key="index"
              ></div>
            </div>
            <div v-else>
              <div v-if="field.name.includes('.*')">
                <div v-if="item && Array.isArray(item[field.name])">
                  <div v-for="(value, index) in item[field.name]" :key="index">
                    {{ value }}
                  </div>
                </div>
                <div v-else>{{ item && item[field.name] }}</div>
              </div>
              <ais-highlight v-else :attribute="field.name" :hit="item" />
            </div>
          </q-item-label>
          <q-item-label
            v-if="item && isImage(item[field.name])"
            caption
            class="img-preview"
          >
            <q-img :src="item[field.name]" fit="contain" class="img-preview" />
          </q-item-label>
        </q-item-section>
        <q-item-section v-if="item && isUrl(item[field.name])" side top>
          <q-item-label>
            <q-btn
              flat
              :href="item[field.name]"
              target="_blank"
              size="sm"
              padding="sm"
              icon="sym_s_open_in_new"
              title="open"
            ></q-btn>
          </q-item-label>
        </q-item-section>
      </q-item>
      <q-separator></q-separator>
      <q-item v-for="field in fieldsNotInSchema" :key="field">
        <q-item-section side class="q-mt-sm text-body2">
          <q-item-label caption> {{ field }} </q-item-label>
          <q-item-label
            class="overflow-hidden text-no-wrap text-ellipsis"
            :title="JSON.stringify(item[field], null, 2)"
          >
            {{ item[field] }}
          </q-item-label>
          <q-item-label v-if="isImage(item[field])" caption class="img-preview">
            <q-img :src="item[field]" fit="contain" class="img-preview" />
          </q-item-label>
        </q-item-section>
        <q-item-section v-if="isUrl(item[field])" side top>
          <q-item-label>
            <q-btn
              flat
              :href="item[field]"
              target="_blank"
              size="sm"
              padding="sm"
              icon="sym_s_open_in_new"
              title="open"
            ></q-btn>
          </q-item-label>
        </q-item-section>
      </q-item>
    </q-list>
    <q-space />
    <q-separator></q-separator>
    <q-item>
      <q-item-section>
        <q-item-label>
          <q-btn
            flat
            size="sm"
            padding="sm"
            @click="editDocument()"
            icon="sym_s_edit"
            title="Edit"
          ></q-btn>
        </q-item-label>
      </q-item-section>
      <q-item-section side>
        <q-item-label>
          <q-btn
            flat
            size="sm"
            padding="sm"
            @click="deleteDocumentById(item.id)"
            icon="sym_s_delete_forever"
            title="Delete"
          ></q-btn>
        </q-item-label>
      </q-item-section>
    </q-item>
  </div>
</template>

<script lang="ts">
import { CollectionSchema } from 'typesense/lib/Typesense/Collection';
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'SearchResultItem',
  props: {
    item: {
      type: Object,
    },
  },
  computed: {
    currentCollection(): CollectionSchema | null {
      return this.$store.state.node.currentCollection;
    },
    fieldsNotInSchema(): string[] {
      if (
        !this.item ||
        !this.currentCollection ||
        !this.currentCollection.fields
      )
        return [];
      const lookup = this.currentCollection.fields
        .map((f) => f.name)
        .concat(['objectID', 'text_match']);
      return Object.keys(this.item).filter(
        (k) => !k.startsWith('_') && !lookup.includes(k)
      );
    },
  },
  methods: {
    isUrl(str: string) {
      if (!str) return false;
      str = String(str);
      return str.startsWith('http://') || str.startsWith('https://');
    },
    isImage(filename: string) {
      if (!filename) return false;
      filename = String(filename);
      const validImageExtensions = [
        'jpg',
        'jpeg',
        'png',
        'gif',
        'bmp',
        'svg',
        'webp',
      ];
      const extension = filename.split('.').pop()?.toLowerCase() || '';
      return validImageExtensions.includes(extension);
    },
    editDocument() {
      //eslint-disable-next-line
      const copyItem: any = {};
      if (!this.item) return;
      Object.keys(this.item).forEach((key) => {
        if (!key.startsWith('_') && !['objectID', 'text_match'].includes(key)) {
          if (!this.item) return;
          copyItem[key] = this.item[key];
        }
      });
      void this.$store.dispatch('node/editDocuments', [
        JSON.parse(JSON.stringify(copyItem)),
      ]);
    },
    deleteDocumentById(id: string) {
      this.$q
        .dialog({
          title: 'Confirm',
          message: `Delete document with id: ${id}?`,
          cancel: true,
          persistent: true,
        })
        .onOk(() => {
          void this.$store.dispatch('node/deleteDocumentById', id);
        });
    },
  },
});
</script>
<style>
.text-ellipsis {
  text-overflow: ellipsis;
}
.search-result-item {
  flex: 1;
}
.img-preview {
  width: 100%;
  height: 150px;
}
.array-field {
  max-height: 150px;
  overflow-y: auto;
  overflow-x: hidden;
}
</style>
