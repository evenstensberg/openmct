<!--
 Open MCT, Copyright (c) 2014-2024, United States Government
 as represented by the Administrator of the National Aeronautics and Space
 Administration. All rights reserved.

 Open MCT is licensed under the Apache License, Version 2.0 (the
 "License"); you may not use this file except in compliance with the License.
 You may obtain a copy of the License at
 http://www.apache.org/licenses/LICENSE-2.0.

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
 WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
 License for the specific language governing permissions and limitations
 under the License.

 Open MCT includes source code licensed under additional open source
 licenses. See the Open Source Licenses file (LICENSES.md) included with
 this source code distribution or the Licensing information page available
 at runtime from the About dialog for additional information.
-->

<template>
  <li
    class="c-recentobjects-listitem c-recentobjects-listitem--object"
    :class="isAlias"
    :aria-label="`${domainObject.name}`"
  >
    <div
      class="c-recentobjects-listitem__type-icon recent-object-icon"
      :class="resultTypeIcon"
    ></div>
    <div class="c-recentobjects-listitem__body">
      <span
        ref="recentObjectName"
        class="c-recentobjects-listitem__title"
        :name="domainObject.name"
        draggable="true"
        @dragstart="dragStart"
        @click.prevent="clickedRecent"
        @mouseover.ctrl="showToolTip"
        @mouseleave="hideToolTip"
      >
        {{ domainObject.name }}
      </span>

      <ObjectPath
        class="c-recentobjects-listitem__object-path"
        :read-only="false"
        :domain-object="domainObject"
        :object-path="objectPath"
      />
    </div>
    <div class="c-recentobjects-listitem__target-button">
      <button
        class="c-icon-button icon-target"
        :aria-label="`Open and scroll to ${domainObject.name}`"
        @click="openAndScrollTo(navigationPath)"
      ></button>
    </div>
  </li>
</template>

<script>
import { PREVIEW_ACTION_KEY } from '@/ui/preview/PreviewAction.js';

import tooltipHelpers from '../../api/tooltips/tooltipMixins.js';
import ObjectPath from '../components/ObjectPath.vue';

export default {
  name: 'RecentObjectsListItem',
  components: {
    ObjectPath
  },
  mixins: [tooltipHelpers],
  inject: ['openmct'],
  props: {
    domainObject: {
      type: Object,
      required: true
    },
    navigationPath: {
      type: String,
      required: true
    },
    objectPath: {
      type: Array,
      required: true
    }
  },
  emits: ['open-and-scroll-to', 'preview-changed'],
  computed: {
    isAlias() {
      return this.openmct.objects.isObjectPathToALink(this.domainObject, this.objectPath)
        ? { 'is-alias': true }
        : undefined;
    },
    resultTypeIcon() {
      return this.openmct.types.get(this.domainObject.type).definition.cssClass;
    }
  },
  mounted() {
    this.previewAction = this.openmct.actions.getAction(PREVIEW_ACTION_KEY);
    this.previewAction.on('isVisible', this.togglePreviewState);
  },
  unmounted() {
    this.previewAction.off('isVisible', this.togglePreviewState);
  },
  methods: {
    clickedRecent(_event) {
      if (this.openmct.editor.isEditing()) {
        this.preview();
      } else {
        this.openmct.router.navigate(`#${this.navigationPath}`);
      }
    },
    togglePreviewState(previewState) {
      this.$emit('preview-changed', previewState);
    },
    preview() {
      if (this.previewAction.appliesTo(this.objectPath)) {
        this.previewAction.invoke(this.objectPath);
      }
    },
    dragStart(event) {
      const navigatedObject = this.openmct.router.path[0];
      const serializedPath = JSON.stringify(this.objectPath);
      const keyString = this.openmct.objects.makeKeyString(this.domainObject.identifier);
      if (this.openmct.composition.checkPolicy(navigatedObject, this.domainObject)) {
        event.dataTransfer.setData(
          'openmct/composable-domain-object',
          JSON.stringify(this.domainObject)
        );
      }

      event.dataTransfer.setData('openmct/domain-object-path', serializedPath);
      event.dataTransfer.setData(`openmct/domain-object/${keyString}`, this.domainObject);
    },
    openAndScrollTo(navigationPath) {
      this.$emit('open-and-scroll-to', navigationPath);
    },
    async showToolTip() {
      const { BELOW } = this.openmct.tooltips.TOOLTIP_LOCATIONS;
      this.buildToolTip(await this.getObjectPath(), BELOW, 'recentObjectName');
    }
  }
};
</script>
