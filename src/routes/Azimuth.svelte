<script lang="ts">
	import { onMount } from 'svelte'
    import Page from './+page.svelte';

	// reactivity value
	let widthPx  = 0;	// canvas サイズ 幅
	let heightPx = 0;	// canvas サイズ 高さ
	let baseX  = 0;		// 方位円の中心X座標
	let baseY  = 0;		// 方位円の中心Y座標
	let radius = 0;		// 方位円の半径(方角先の長さ)

	// /**
	//  * 1ドット　描画
	//  * @param context 描画対象2Dコンテキスト
	//  * @param x 開始位置x
	//  * @param y 開始位置y
	//  * @param style
	//  */
	// const dot = (context: CanvasRenderingContext2D, x: number, y: number, style: string = 'cyan') => {
	// 	context.fillStyle = style;
	// 	context.fillRect(x, y, 1, 1);
	// }

	/**
	 * ライン 描画
	 * @param context 描画対象2Dコンテキスト
	 * @param x1 開始位置x
	 * @param y1 開始位置y
	 * @param x2 終了位置x
	 * @param y2 終了位置y
	 * @param style 描画スタイル
	 */
	const line = (context: CanvasRenderingContext2D, x1: number, y1: number, x2: number, y2: number, style: string = 'red') => {
		context.strokeStyle = style;
		context.beginPath();
		context.moveTo(x1, y1);
		context.lineTo(x2, y2);
		context.stroke();
//		context.closePath();
	}

	/**
	 * 角度から単位円(半径=1の円周)上の座標を取得する。
	 *
	 * @param deg　角度(度数)
	 * @return object {x:, y:} 単位円上での座標
	 */
	const calcCircleAngle = (deg: number): object => {
		const rad = deg * Math.PI / 180;
		return {x: Math.sin(rad), y: Math.cos(rad)}
	}

	/**
	 * 方位のラインと方位角を描画
	 * @param context
	 * @param x 単位円上での描画位置X座標
	 * @param y 単位円上での描画位置Y座標
	 */
	const azimuthLine = (context: CanvasRenderingContext2D, x: number, y: number, degStr: string) => {
		x = x * radius;
		y = y * radius;

		// ライン描画
		line(context, baseX, baseY, baseX + x, baseY + y * -1, '#f00');

		// 方位角描画
		let fontPx = heightPx / 25
		if (heightPx < 300) {
			fontPx = 12;
		}
		context.font = "bold " + fontPx + "px serif";
		let	textX = 0;
		let	textY = 0;
		{
			// 表示位置の微調整
			let textWidth = context.measureText(degStr).width;
			if ((-3 < x) && (x < 3)) {
				textX = baseX + x - textWidth / 2 + 1;
			} else if (x < 0) {
				textX = baseX + x - textWidth;
			} else {
				textX = baseX + x;
			}
			if ((-3 < y) && (y < 3)) {
				textY = baseY + y * - 1 + fontPx / 3;
			} else if (y < 0) {
				textY = baseY + y * - 1 + fontPx;
			} else {
				textY = baseY + y * - 1 - fontPx / 3;
			}
		}
		context.fillStyle = "#0000ff";
  	context.fillText(degStr, textX, textY);
	}

	const drawAzimuthLines = (context: CanvasRenderingContext2D) => {
    context.fillStyle = '#f0f0f0';
    context.fillRect(0, 0, widthPx, heightPx);

		let obj = calcCircleAngle(0);	azimuthLine(context, obj.x, obj.y, "0");
		obj = calcCircleAngle(22.5);	azimuthLine(context, obj.x, obj.y, "22.5");
		obj = calcCircleAngle(45);		azimuthLine(context, obj.x, obj.y, "45");
		obj = calcCircleAngle(67.5);	azimuthLine(context, obj.x, obj.y, "67.5");
		obj = calcCircleAngle(90);		azimuthLine(context, obj.x, obj.y, "90");
		obj = calcCircleAngle(112.5);	azimuthLine(context, obj.x, obj.y, "112.5");
		obj = calcCircleAngle(135);		azimuthLine(context, obj.x, obj.y, "135");
		obj = calcCircleAngle(157.5);	azimuthLine(context, obj.x, obj.y, "157.5");
		obj = calcCircleAngle(180);		azimuthLine(context, obj.x, obj.y, "180");
		obj = calcCircleAngle(202.5);	azimuthLine(context, obj.x, obj.y, "202.5");
		obj = calcCircleAngle(225);		azimuthLine(context, obj.x, obj.y, "225");
		obj = calcCircleAngle(247.5);	azimuthLine(context, obj.x, obj.y, "247.5");
		obj = calcCircleAngle(270);		azimuthLine(context, obj.x, obj.y, "270");
		obj = calcCircleAngle(292.5);	azimuthLine(context, obj.x, obj.y, "292.5");
		obj = calcCircleAngle(315);		azimuthLine(context, obj.x, obj.y, "315");
		obj = calcCircleAngle(337.5);	azimuthLine(context, obj.x, obj.y, "337.5");
	}

	/**
	 * 方位円の座標(x, y)の方位角(deg)を求める
	 *
	 * @param x x座標
	 * @param y y座標
	 * @return 方位角 (0〜359°)
	 */
	const mouseToDeg = (x: number, y: number): number => {
		let a = x - baseX;
		let b = y - baseY;
		let deg = Math.atan(b / a) * 180 / Math.PI;
		if (0 < a)			deg = deg + 90;
		else if (0 > a)	deg = deg + 270;

		return deg;
	}

	//--------------------
	// events

	onMount(() => {
		onResize();
		window.addEventListener('resize', onResize);
	})

	function onResize() {
  	const canvas: HTMLCanvasElement = document.getElementById("azimuth-canvas");
    const context: CanvasRenderingContext2D = canvas.getContext("2d");

		widthPx  = window.innerWidth - 10;
		heightPx = window.innerHeight * 0.80;

		canvas.width  = widthPx;
		canvas.height = heightPx;

		baseX = widthPx / 2;
		baseY = heightPx / 2;
		radius = heightPx / 2 * 0.8;

		drawAzimuthLines(context);
	}

	const handleMove = (({offsetX: x, offsetY: y}) => {
		const deg: number = mouseToDeg(x, y);
		const elem = document.getElementById("mouse-deg").innerText = "" + parseInt(deg) + "°";
	})
	

</script>

<div>
	方位角：<span id="mouse-deg">---</span>
</div>
<canvas
	id="azimuth-canvas"
	on:mousemove={handleMove}
/>

<style>
</style>
