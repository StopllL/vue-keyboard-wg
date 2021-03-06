<script>
	export default {
		name: 'keyboard',
		
		props: {
			value: {
				type: String,
				default: ''
			},
			layouts: {
				type: [String, Array],
				default:function(){
					return ['qwertyuiop|asdfghjkl|{upper}zxcvbnm{backspace}|{123:goto:1}{space:space}.{Go:custom}','1234567890|!@#$\"&*()|?^/[]:;%{backspace}|{ABC:goto:0}{space:space}.{custom:custom}']
				}
			},
			maxlength: {
				type: Number,
				default: 0,
				validator(value) { return value >= 0; }
			},
			pattern: {
				type: String,
				default: null
			}
		},

		data() {
			return {
				layout: 0,
				isUpper:false
			};
		},

		computed: {
			/**
			 * Whether or not the keyboard input has hit its maximum length.
			 * @returns {Boolean}
			 */
			full() {
				return this.maxlength > 0 && this.value.length >= this.maxlength;
			},

			/**
			 * Whether or not the keyboard input is empty.
			 * @return {Boolean}
			 */
			empty() {
				return this.value.length === 0;
			},

			/**
			 * Returns an array of buttons to render in the component.
			 * @returns {Array[]}
			 */
			buttons() {
				let lines = (Array.isArray(this.layouts) ? this.layouts : [this.layouts])[this.layout].split('|');
				var upper = this.$data.isUpper;
				return lines.map(chars => {
					let stream = chars.split('');
					let buttons = [];
					let token = null;

					stream.forEach(char => {
						if (char === '{') {
							token = '';
						} else if (char === '}') {
							let iconclass = '';
							let command = token.split(':');
							let text = command.length > 1 ? command[0] : '';
							if(!text){
								switch(command[0]){
									case 'backspace' : iconclass = "iconfont icon-shuzijianpanshanchu";
													   break;
								   	case 'upper' : iconclass = "iconfont icon-jianpanqiehuandaxiaoxie"
								}
							}
							let action = command.length > 1 ? command[1] : command[0];
							let args = command.length > 2 ? command[2] : null;
							let method = null;

							if (['append', 'backspace', 'space', 'clear', 'goto', 'upper'].indexOf(action) >= 0) method = this[action].bind(this, args);
							else method = this.$emit.bind(this, action, this);

							buttons.push({
								type: command.length > 1 ? 'action' : 'action ' + iconclass,
								action: { name: action.replace(/\s+/g, '-').toLowerCase(), callable: method },
								value: text,
								args: args
							});
							
							token = null;
						} else {
							if (token !== null) {
								token += char;
							} else {
								buttons.push({
									type: 'char',
									action: { name: null, callable: this.append.bind(this, char) },
									value: upper ? char.toUpperCase() :char,
									args: null
								});
							}
						}
					});
					return buttons;
				});
			},

			/**
			 * Whether or not the current value matches the regex provided to pattern. Always
			 * returns true if no pattern was provided.
			 * @returns {Boolean}
			 */
			valid() {
				return !this.pattern || this.value.match(new RegExp(this.pattern));
			}
		},

		methods: {
			/**
			 * Mutates the keyboard value to a new value.
			 * @param {String} value The new value.
			 */
			mutate(value) {
				if (this.maxlength > 0) {
					value = value.slice(0, this.maxlength);
				}
				this.$emit('input', value);
			},

			/**
			 * Appends a new value to the end of the current keyboard value.
			 * @param {String} char The character(s) to append.
			 */
			append(char) {
				if(this.isUpper){
					this.mutate(this.value + char.toUpperCase())
				}else{
					this.mutate(this.value + char);
				}
			},

			/**
			 * Remove the last character from the current keyboard value.
			 */
			backspace() {
				this.mutate(this.value.slice(0, this.value.length - 1));
			},

			/**
			 * Add one whitespace character to the current keyboard value.
			 */
			space() {
				this.append(' ');
			},

			/**
			 * Go to a new layout.
			 * @param {Number} The layout index.
			 */
			goto(layout) {
				if (Array.isArray(this.layouts)) {
					if (layout >= 0 && layout < this.layouts.length) {
						this.layout = layout;
					} else {
						throw new Error('The requested layout does not exist.');
					}
				} else {
					throw new Error('A single non-array layout was provided.');
				}
			},
			upper(){
				this.isUpper = !this.isUpper;
			},
			/**
			 * Clear the entire keyboard value.
			 */
			clear() {
				this.mutate('');
			}
		}
	}
</script>

<template>
	<aside class="vue-keyboard" role="application" :class="{ full: full, empty: empty, valid: valid, invalid: !valid }" :data-value="value" :data-layout="layout">
		<div role="row" class="row" v-for="row in buttons" :data-keys="row.length">
			<button
				v-for="btn in row"
				role="button"
				:class="btn.type"
				:data-args="btn.args"
				:data-text="btn.value"
				:data-action="btn.action.name"
				@click.prevent="btn.action.callable"
			>{{ btn.value }}</button>
		</div>
	</aside>
</template>

<style lang="scss" scoped>
	@font-face {font-family: "iconfont";
      src: url('iconfont.eot'); /* IE9*/
      src: url('iconfont.eot#iefix') format('embedded-opentype'), /* IE6-IE8 */
      url('iconfont.woff') format('woff'), /* chrome, firefox */
      url('iconfont.ttf') format('truetype'), /* chrome, firefox, opera, Safari, Android, iOS 4.2+*/
      url('iconfont.svg#iconfont') format('svg'); /* iOS 4.1- */
    }

    .iconfont {
      font-family:"iconfont" !important;
      font-size:16px;
      font-style:normal;
      -webkit-font-smoothing: antialiased;
      -webkit-text-stroke-width: 0.2px;
      -moz-osx-font-smoothing: grayscale;
      color:red;
    }
    .icon-jianpanqiehuandaxiaoxie:before { content: "\e71c"; }

	.icon-jianpanshanchu:before { content: "\e71f"; }

	.icon-shuzijianpanshanchu:before { content: "\e74b"; }
	.vue-keyboard {
		position:fixed;
		bottom:0;
		left:0;
		width:100%;
		font-size:0.4rem;
		line-height:0.4rem;
		background:#d1d5da;
		.row {
			padding: 2px 0;
			text-align: center;
		}

		button {
			border: none;
			box-shadow:0px 3px 4px #888b8e;
			outline: none;
			padding: 2px 0.1rem;
			min-width: 1.1rem;
			min-height:0.6rem;
			margin: 2px 4px;
			background: #EEE;
			color: #666;
			cursor: pointer;
			font-family: inherit;
			font-size: inherit;
			line-height:inherit;
			border-radius: 0.1rem;
			&:hover {
				background: #E0E0E0;
			}
			&:active {
				background: #777;
				color: #FFF;
				box-shadow: inset 0 1px 4px rgba(#000, 0.1);
			}

			&[data-action="space"] {
				padding: 4px 1rem;
			}
		}
		.action{
			background: #acb2bc;
		}
	}
</style>