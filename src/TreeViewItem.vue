<template>
	<div class="tree-view-item">
	<div v-if="isObject(data)" class="tree-view-item-leaf">
		<div class="tree-view-item-node" @click.stop="toggleOpen()">
		<span :class="{opened: isOpen()}" class="tree-view-item-key tree-view-item-key-with-chevron">{{getKey(data)}}</span>
		<span class="tree-view-item-hint" v-show="!isOpen() && data.children.length === 1">{{data.children.length}} property</span>
		<span class="tree-view-item-hint" v-show="!isOpen() && data.children.length !== 1">{{data.children.length}} properties</span>
		</div>
		<div v-if="!limitRenderDepth || isOpen()">
		<tree-view-item
			:key="getKey(child)"
			:max-depth="maxDepth"
			:current-depth="currentDepth+1"
			v-show="isOpen()"
			v-for="child in data.children"
			:data="child"
			:modifiable="modifiable"
			:link="link"
			:limit-render-depth="limitRenderDepth"
			@change-data="onChangeData"/>
		</div>
	</div>
	<div v-if="isArray(data)" class="tree-view-item-leaf">
		<div class="tree-view-item-node" @click.stop="toggleOpen()">
		<span :class="{opened: isOpen()}" class="tree-view-item-key tree-view-item-key-with-chevron">{{getKey(data)}}</span>
		<span class="tree-view-item-hint" v-show="!isOpen() && data.children.length === 1">{{data.children.length}} item</span>
		<span class="tree-view-item-hint" v-show="!isOpen() && data.children.length !== 1">{{data.children.length}} items</span>
		</div>
		<div v-if="!limitRenderDepth || isOpen()">
		<tree-view-item
			:key="getKey(child)"
			:max-depth="maxDepth"
			:current-depth="currentDepth+1"
			v-show="isOpen()"
			v-for="child in data.children"
			:data="child"
			:modifiable="modifiable"
			:link="link"
			:limit-render-depth="limitRenderDepth"
			@change-data="onChangeData"/>
		</div>
	</div>
	<tree-view-item-value
		v-if="isValue(data)"
		class="tree-view-item-leaf"
		:key-string="getKey(data)"
		:data="data.value"
		:modifiable="modifiable"
		:link="link"
		@change-data="onChangeData"/>
	</div>
</template>

<script>
import _ from 'lodash';
import TreeViewItemValue from './TreeViewItemValue.vue';

export default {
	components: {
		TreeViewItemValue
	},
	name: 'tree-view-item',
	props: ['data', 'max-depth', 'current-depth', 'modifiable', 'link', 'limit-render-depth'],
	data() {
		return {
			open: this.currentDepth < this.maxDepth
		};
	},

	watch: {
		data: {
			handler(newValue) {
				// Open the "open" key in JSON view if "isOpen" is truthy.
				if (newValue?.isOpen) this.open = true;
				// Close the "open" key in JSON view if "isOpen" is explicitly set to false.
				else if (newValue?.isOpen === false) this.open = false
			},
			immediate: true
		}
	},
	methods: {
		isOpen() {
			return this.open;
		},
		toggleOpen() {
			this.open = !this.open;
		},
		isObject(value) {
			return value.type === 'object';
		},
		isArray(value) {
			return value.type === 'array';
		},
		isValue(value) {
			return value.type === 'value';
		},
		getKey(value) {
			if (_.isInteger(value.key)) {
				return `${value.key}:`;
			}
			return `"${value.key}":`;

		},
		isRootObject(value = this.data) {
			return value.isRoot;
		},
		onChangeData(path, value) {
			const newPath = _.concat(this.data.key, path);
			this.$emit('change-data', newPath, value);
		}
	}
};
</script>

<style scoped>
.tree-view-item {
	font-family: monaco, monospace;
	font-size: 14px;
	margin-left: 18px;
}

.tree-view-item-node {
	cursor: pointer;
	position: relative;
	white-space: nowrap;
}

.tree-view-item-leaf {
	white-space: nowrap;
}

.tree-view-item-key {
	font-weight: bold;
}

.tree-view-item-key-with-chevron {
	padding-left: 14px;
}

.tree-view-item-key-with-chevron.opened::before {
	top:4px;
	transform: rotate(90deg);
	-webkit-transform: rotate(90deg);
}

.tree-view-item-key-with-chevron::before {
	color: #444;
	content: '\25b6';
	font-size: 10px;
	left: 1px;
	position: absolute;
	top: 3px;
	transition: -webkit-transform .1s ease;
	transition: transform .1s ease;
	transition: transform .1s ease, -webkit-transform .1s ease;
	-webkit-transition: -webkit-transform .1s ease;
}

.tree-view-item-hint {
	color: #ccc
}
</style>
