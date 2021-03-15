<template>
  <aside>
    <span v-if="canPublishNb || canPublishNn">
      <button v-if="isPublishedNb" @click="unpublish('nb')">Arkiver nb</button>
      <button v-else-if="canPublishNb" @click="publish('nb')">
        Publiser nb
      </button>
      <br />
      <button v-if="isPublishedNn" @click="unpublish('nn')">Arkiver nn</button>
      <button v-else-if="canPublishNn" @click="publish('nn')">
        Publiser nn
      </button>
    </span>

    <button v-else-if="stateHasChanges" @click="save">Lagre</button>
    <button v-else disabled="true">Lagre</button>

    <span v-if="canPublishNb || canPublishNn">
      <div v-if="isPublishedNb">
        Innholdet på bokmål er publisert og kan arkiveres
      </div>
      <div v-else-if="canPublishNb">
        Draften er validert på bokmål og kan publiseres
      </div>
      <div v-if="isPublishedNn">
        Innholdet på nynorsk er publisert og kan arkiveres
      </div>
      <div v-else-if="canPublishNn">
        Draften er validert på nynorsk og kan publiseres
      </div>
    </span>
    <div v-if="canPublish">Det er ingen endringer i draften.</div>
    <div v-else-if="stateHasChanges">Draften har ulagrede endringer</div>
  </aside>
  <main>
    <div>Tittel</div>
    nb <input type="text" v-model="content.title.nb" />{{
      isTitleNbValid ? "🟩" : "🟥"
    }}
    <br />
    nn <input type="text" v-model="content.title.nn" />{{
      isTitleNnValid ? "🟩" : "🟥"
    }}
  </main>
  <div class="draft">
    Lagret draft
    <pre>{{ draft }}</pre>
  </div>
  <div class="stage">
    Vises i Stage
    <pre>{{ validDraft }}</pre>
  </div>
  <div class="prod">
    Vises i Prod
    <pre>{{ published }}</pre>
  </div>
</template>

<script setup>
import { computed, ref } from "vue";

const content = ref({
  title: {
    nb: "",
    nn: "",
  },
});
const isTitleNbValid = computed(() => content.value.title.nb.length > 1);
const isTitleNnValid = computed(() => content.value.title.nn.length > 1);

const draft = ref({});
const validDraft = ref({});
const save = () => {
  draft.value = copy(content.value);
  validDraft.value = copy(content.value);
  if (!isTitleNbValid.value) {
    delete validDraft.value.title.nb;
  }
  if (!isTitleNnValid.value) {
    delete validDraft.value.title.nn;
  }
  if (!isTitleNbValid.value && !isTitleNnValid.value)
    delete validDraft.value.title;
};
const stateHasChanges = computed(
  () => jsonEqual(content.value, draft.value) === false
);

const published = ref({});
const publish = (cultureCode) => {
  let title = published.value.title || {};
  title[cultureCode] = draft.value.title[cultureCode];
  published.value = { title };
};
const unpublish = (cultureCode) => {
  let title = published.value.title || {};
  delete title[cultureCode];
  published.value = { title };
  if (!title.nb && !title.nn) delete published.value.title;
};
const canPublishNb = computed(
  () => stateHasChanges.value === false && isTitleNbValid.value === true
);
const canPublishNn = computed(
  () => stateHasChanges.value === false && isTitleNnValid.value === true
);

const isPublishedNb = computed(
  () => published.value.title?.nb === draft.value.title?.nb
);
const isPublishedNn = computed(
  () => published.value.title?.nn === draft.value.title?.nn
);

const jsonEqual = (a, b) => JSON.stringify(a) === JSON.stringify(b);
const copy = (x) => JSON.parse(JSON.stringify(x));
</script>

<style>
#app {
  display: grid;
  grid-template: 1fr 3fr / 1fr 1fr 1fr;
}

aside {
  grid-row: 1;
  grid-column: 3;
}

button {
  font-size: 2em;
}

main {
  align-self: center;
  grid-row: 1;
  grid-column: 2;
  min-height: 22vh;
}

.draft {
  grid-row: 2;
  grid-column: 1;
}
.stage {
  grid-row: 2;
  grid-column: 2;
}
.prod {
  grid-row: 2;
  grid-column: 3;
}
</style>