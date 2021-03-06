<template lang="pug">
.sleeve(
	tabindex="0"
	:class="quad.yPos, quad.xPos, quad.edge, quad.middle"
	)
	.card(ref="card")
		transition(name="flip")
			HexaFace(
				v-if="!cfg.texty"
				class="face face--front"
				:kingwen="hex.kingwen"
				@close="$emit('close')"
				tabindex="-1"
				)
				template(#top)
					router-link.mark(to="/" v-if="mark") {{mark}}
					HexaNames(
						:names="hex.names"
						:kingwen="hex.kingwen"
						:octal="hex.octal"
						)
				template(#bottom)
					h3.mrg.mrg1.y.font.head.xl The Image
					.images.left
						pre.image.text.sd.fine {{ hex.images }}
					.cross.horiz.flex
						.flex.col.mid.less
							LineGlyph( :glyph="hex.hexagram" size="x6l" )
							.binary.font.sm(v-show="cfg.turny") {{ hex.binary.slice(2) }}
							.decimal.font.sm(v-show="cfg.turny") {{ parseInt(hex.binary.slice(2), 2) + ' of 64' }}
							.kingwen.font.sm(v-show="!cfg.turny")
								span King Wen 
								span {{ hex.kingwen }}
							.octal.font.sm(v-show="!cfg.turny")
								span Octal 
								span {{ hex.octal }}
						.liangua.spread.flex.col.even
							OneGua(
								:gua="hex.trigramPair.above"
								size="x4l"
								charSize="md"
								above
								)
								.pos-name.font.sm above
							OneGua(
								:gua="hex.trigramPair.below"
								size="x4l"
								charSize="md"
								below
								)
								.pos-name.font.sm below
			HexaFace(
				v-else
				class="face face--back"
				:kingwen="hex.kingwen"
				@close="$emit('close')"
				tabindex="-1"
				)
				template(#top)
					router-link.mark(to="/" v-if="mark") {{mark}}
					Spinnable
						LineGlyph(
							:glyph="hex.hexagram"
							size="x7l"
							noturn
							)
					h2.yingyu.head.x2l {{ hex.names.english }}
					pre.judgment.text.sd.fine {{ hex.judgment }}
				template(#bottom)
					h3.mrg.mrg1.y.font.head.xl The Lines
					.lines.pad.pad1.y
						IconBase.line(
							v-for="digit in hex.binary.slice(2)"
							:key="$symbolize(digit)"
							size="36"
							width="20"
							)
							component(
								v-if="digit === '0'"
								:is="`Icon6`")
							component(
								v-if="digit === '1'"
								:is="`Icon9`")
					ChangingLines(
						:hex="hex"
						:liney="liney")
</template>
<script lang="ts">
import {defineComponent, PropType, ref, reactive, toRefs, onMounted} from 'vue'
import {defHex, Quad, defQuad, Hexagram} from '../schema'
import {useSwipeable} from '../composables/swipeable'
import {cfg, tog} from '../store'
import OneGua from './OneGua.vue'
import Icon6 from '../icons/Icon6.vue'
import Icon7 from '../icons/Icon7.vue'
import Icon8 from '../icons/Icon8.vue'
import Icon9 from '../icons/Icon9.vue'
import IconBase from '../icons/IconBase.vue'
import HexaFace from './HexaFace.vue'
import HanziChar from './HanziChar.vue'
import LineGlyph from './LineGlyph.vue'
import Turnable from './Turnable.vue'
import Spinnable from './Spinnable.vue'
import HexaNames from './HexaNames.vue'
import ChangingLines from './ChangingLines.vue'

export default defineComponent({
	name: 'HexaCard',
	components: {
		Icon6,
		Icon7,
		Icon8,
		Icon9,
		OneGua,
		IconBase,
		HexaFace,
		HanziChar,
		LineGlyph,
		Turnable,
		Spinnable,
		HexaNames,
		ChangingLines,
	},
	props: {
		hex: {
			type: Object as PropType<Hexagram>,
			default: defHex,
		},
		quad: {
			type: Object as PropType<Quad>,
			default: defQuad,
		},
		liney: Boolean,
		mark: {
			type: String,
			default: '',
		},
	},
	emits: ['close'],
	setup() {
		const {handleSwipeStart, handleSwipeEnd} = useSwipeable()
		const rx = reactive({
			card: ref<HTMLElement>(),
			interpShown: false,
		})

		onMounted(() => {
			const gestureZone = rx.card
			if (!gestureZone) return
			gestureZone.addEventListener('touchstart', handleSwipeStart, false)
			gestureZone.addEventListener('touchend', handleSwipeEnd, false)
		})

		return {
			tog,
			cfg,
			...toRefs(rx),
		}
	},
})
</script>

<style lang="postcss" scoped>
.mark {
	font-size: 1rem;
	@supports (font-variation-settings: normal) {
		font-family: 'QuicksandVariable';
		font-variation-settings: 'wght' 666;
	}
	border-radius: 100%;
	padding: 0.25em;
	color: var(--flair);
	margin-top: -2.75em;
	text-align: center;
	text-decoration: none;
	transition-property: color, text-decoration;
	transition-duration: var(--beat);
	&:hover {
		color: var(--link);
		text-decoration: dotted underline;
	}
}

.card .hexaglyph.font {
	margin: 0.5rem 0;
}

.sleeve {
	position: fixed;
	top: 10%;
	right: 5%;
	left: 5%;
	overflow: visible;
	text-align: left;
	z-index: 20;
}

.card {
	position: absolute;
	z-index: 21;
	font-size: 1.125em;
}

.sleeve,
.card {
	width: 92vw;
	height: 22rem;
	max-width: 27rem;
	max-height: 36rem;
	margin: auto;

	@media (min-width: 36rem) {
		width: 27rem;
	}

	@media (min-height: 36rem) {
		height: 32rem;
	}

	@media (min-width: 48rem) and (min-height: 36rem) {
		font-size: 1.125em;
		width: 32rem;
		height: 40rem;
	}

	@media (min-height: 48rem) and (min-width: 36rem) {
		height: 36rem;
	}
}

hr.divider {
	margin: 2rem auto 0;
}

.active .sleeve {
	z-index: 25;
}

.active .sleeve:focus-within {
	z-index: 26;
}

.face {
	color: var(--ink);
	background-color: var(--paper);
}

.cross.horiz {
	display: flex;
	flex-direction: row;
	align-items: flex-start;
	min-width: 16em;
	padding-bottom: 1rem;
}

@media (min-width: 36rem) and (min-height: 36rem) {
	.cross.horiz {
		font-size: 125%;
	}
}

pre.judgment,
pre.image {
	line-height: var(--leading);
}

@media (orientation: landscape) and (max-height: 35.92rem) {
	.sleeve {
		left: 40%;
		right: 5%;
		top: 0;
		bottom: 5%;
	}

	.sleeve,
	.card {
		width: 24em;
		height: 100vh;
	}
}

@media (min-width: 36rem) and (min-height: 36rem) {
	.sleeve {
		position: absolute;
		margin: auto;
		left: 50%;
		transform: translate3d(-50%, 0, 0);
	}

	.sleeve.top {
		top: calc(100% + 1.25rem);
		bottom: unset;
	}

	.sleeve.bottom {
		bottom: calc(100% + 1.25rem);
		top: unset;
	}

	.sleeve.top.middle {
		top: -60%;
	}

	.sleeve.bottom.middle {
		bottom: -60%;
	}

	.sleeve.middle.right {
		left: 100%;
	}

	.sleeve.middle.left {
		right: 100%;
	}

	.sleeve.right.edge {
		right: -100%;
		left: 20%;
	}

	.sleeve.left.edge {
		left: 100%;
	}

	.sleeve.left.extreme {
		left: 200%;
	}

	.sleeve.right.extreme {
		left: -100%;
	}

	.sleeve.left.edge.middle {
		left: 100%;
	}
}
.pos-name {
	margin-top: -1em;
}
</style>
