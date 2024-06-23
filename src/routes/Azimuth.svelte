<script lang="ts">
	import { onMount } from 'svelte'
	export let drawOrthAngle = false;

	// reactivity values
	let widthPx  = 0;	// canvas サイズ 幅
	let heightPx = 0;	// canvas サイズ 高さ
	let baseX  = 0;		// 方位円の中心X座標
	let baseY  = 0;		// 方位円の中心Y座標
	let radius = 0;		// 方位円の半径(方角先の長さ)

	let mouseDownFlag = false;
	let mouseDownX = 0;
	let mouseDownY = 0;

	interface XYByDeg {
		x: number;
		y: number;
	}

	let orthogonalAngleDeg1 = "";
	let orthogonalAngleDeg2 = "";

	/**
	 * 方位角(deg)から単位円(半径=1の円周)上の座標を取得する
	 * @param deg　角度(度数)
	 * @return object {x:, y:} 単位円上での座標
	 */
	const getXYByDeg = (deg: number): XYByDeg => {
		const rad = deg * Math.PI / 180;
		return {x: Math.sin(rad), y: Math.cos(rad)}
	}

	/**
	 * 方位円上の座標(x, y)から方位角(deg)を取得する
	 * @param x x座標
	 * @param y y座標
	 * @return 方位角 (0〜359°)
	 */
	const getDegByXY = (x: number, y: number): number => {
		let a = x - baseX;
		let b = y - baseY;
		let deg = Math.atan(b / a) * 180 / Math.PI;
		if (0 < a)			deg = deg + 90;
		else if (0 > a)	deg = deg + 270;
		return deg;
	}

	/**
	 * ライン 描画
	 * @param context 描画対象2Dコンテキスト
	 * @param x1 開始位置x
	 * @param y1 開始位置y
	 * @param x2 終了位置x
	 * @param y2 終了位置y
	 * @param style 描画スタイル
	 */
	const line = (context: CanvasRenderingContext2D, x1: number, y1: number, x2: number, y2: number, width: number = 1, style: string = 'red') => {
		context.strokeStyle = style;
		context.beginPath();
		context.moveTo(x1, y1);
		context.lineTo(x2, y2);
		context.lineWidth = width;
		context.stroke();
//		context.closePath();
	}

	/**
	 * 方位ラインと方位角文字列を描画
	 * @param context canvasの2D context
	 * @param x 単位円上での描画位置X座標
	 * @param y 単位円上での描画位置Y座標
	 * @param degStr 方位角文字列
	 */
	const azimuthLine = (context: CanvasRenderingContext2D, x: number, y: number, degStr: string) => {
		x = x * radius;
		y = y * radius;

		// 方位ライン描画
		line(context, baseX, baseY, baseX + x, baseY + y * -1, 1, '#f00');

		// 方位角文字列描画
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
		context.fillStyle = "#00f";
  	context.fillText(degStr, textX, textY);
	}

	/**
	 * 方位角の線を描画する
	 * @param context canvasの2D context
	 */
	const drawAzimuthLines = (context: CanvasRenderingContext2D) => {
    context.fillStyle = '#f0f0f0';
    context.fillRect(0, 0, widthPx, heightPx);

		let obj: XYByDeg = getXYByDeg(0);	azimuthLine(context, obj.x, obj.y, "0");
		obj = getXYByDeg(22.5);						azimuthLine(context, obj.x, obj.y, "22.5");
		obj = getXYByDeg(45);							azimuthLine(context, obj.x, obj.y, "45");
		obj = getXYByDeg(67.5);						azimuthLine(context, obj.x, obj.y, "67.5");
		obj = getXYByDeg(90);							azimuthLine(context, obj.x, obj.y, "90");
		obj = getXYByDeg(112.5);					azimuthLine(context, obj.x, obj.y, "112.5");
		obj = getXYByDeg(135);						azimuthLine(context, obj.x, obj.y, "135");
		obj = getXYByDeg(157.5);					azimuthLine(context, obj.x, obj.y, "157.5");
		obj = getXYByDeg(180);						azimuthLine(context, obj.x, obj.y, "180");
		obj = getXYByDeg(202.5);					azimuthLine(context, obj.x, obj.y, "202.5");
		obj = getXYByDeg(225);						azimuthLine(context, obj.x, obj.y, "225");
		obj = getXYByDeg(247.5);					azimuthLine(context, obj.x, obj.y, "247.5");
		obj = getXYByDeg(270);						azimuthLine(context, obj.x, obj.y, "270");
		obj = getXYByDeg(292.5);					azimuthLine(context, obj.x, obj.y, "292.5");
		obj = getXYByDeg(315);						azimuthLine(context, obj.x, obj.y, "315");
		obj = getXYByDeg(337.5);					azimuthLine(context, obj.x, obj.y, "337.5");
	}

	/**
	 * 方位角補助線
	 */
	const followLine = () => {

		if ((mouseDownX == 0) && (mouseDownY == 0))
			return;

		let x = mouseDownX;
		let y = mouseDownY;

		const deg: string = getDegByXY(x, y).toFixed(1);

  	const canvas: HTMLCanvasElement = document.getElementById("azimuth-canvas") as HTMLCanvasElement;
    const context: CanvasRenderingContext2D = canvas.getContext("2d")!;
		{
			line(context, baseX, baseY, x, y, 4, '#0ff');

			context.fillStyle = "#0ff";
			const circle = new Path2D();
			circle.arc(x, y, 4, 0, 2 * Math.PI);
			context.fill(circle);

			let fontPx = heightPx / 25
			if (heightPx < 300) {
				fontPx = 12;
			}
			context.font = "italic " + fontPx + "px serif";
			context.fillStyle = "#00f";
			context.fillText(deg, x + 4, y - 4);
		}

		if (drawOrthAngle) {	// 直行LINE表示
			let mx1 = baseX + (baseY - y);
			let my1 = baseY - (baseX - x);
			let mx2 = baseX - (baseY - y);
			let my2 = baseY + (baseX - x);
			line(context, mx1, my1, mx2, my2, 4, '#0ff');

			// 直行角算出(deg)
			orthogonalAngleDeg1 = getDegByXY(mx1, my1).toFixed(1);
			orthogonalAngleDeg2 = getDegByXY(mx2, my2).toFixed(1);

			// document.getElementById("orthogonal-deg1")!.innerText = orthogonalAngleDeg1 + "°";
			// document.getElementById("orthogonal-deg2")!.innerText = orthogonalAngleDeg2 + "°";


			let fontPx = heightPx / 40
			if (heightPx < 300) {
				fontPx = 10;
			}
			context.font = "italic " + fontPx + "px serif";
			context.fillStyle = "#00f";
			context.fillText(orthogonalAngleDeg1, mx1 + 4, my1 - 4);
			context.fillText(orthogonalAngleDeg2, mx2 + 4, my2 - 4);
		}
	}

	//--------------------
	// events
	/**
	 * マウント時の処理
	 */
	onMount(() => {
		onRedraw();
		window.addEventListener('resize', onResize);
	})

	/**
	 * ウィンドウリサイズ時の処理
	 */
	const onResize = () => {
		onRedraw();
	}

	/**
	 * 再描画
	 */
	const onRedraw = () => {
  	const canvas: HTMLCanvasElement = document.getElementById("azimuth-canvas") as HTMLCanvasElement;
    const context: CanvasRenderingContext2D = canvas.getContext("2d")!;

		widthPx  = window.innerWidth - 10;
		heightPx = window.innerHeight * 0.80;

		canvas.width  = widthPx;
		canvas.height = heightPx;

		baseX = widthPx / 2;
		baseY = heightPx / 2;
		if (widthPx > heightPx)	radius = heightPx / 2 * 0.8;
		else										radius = widthPx / 2 * 0.6;

		drawAzimuthLines(context);
	}

	//--------------------
	// event handlers

	const handleMouseMove = (e: MouseEvent) => {
		let x = e.offsetX;
		let y = e.offsetY;
		const deg: string = getDegByXY(x, y).toFixed(1);
		document.getElementById("mouse-deg")!.innerText = deg + "°";

		if (mouseDownFlag) {
			mouseDownX = e.offsetX;
			mouseDownY = e.offsetY;
			onRedraw();
			followLine();
		}
	}

	const handleMouseDown = (e: MouseEvent) => {
		if (e.button == 0) {
			mouseDownFlag = true;
			mouseDownX = e.offsetX;
			mouseDownY = e.offsetY;
		}

		onRedraw();
		followLine();
	}

	const handleMouseUp = (e: MouseEvent) => {
		mouseDownFlag = false;
		mouseDownX = e.offsetX;
		mouseDownY = e.offsetY;
	}

	const handleMouseOut = (e: MouseEvent) => {
		mouseDownFlag = false;
		mouseDownX = e.offsetX;
		mouseDownY = e.offsetY;
	}

</script>

<div>
	方位角：<span id="mouse-deg">---</span>
{#if drawOrthAngle}
	<!-- (直行角：<span id="orthogonal-deg1">---</span> / <span id="orthogonal-deg2">---</span>) -->
{/if}
</div>

<canvas
	id="azimuth-canvas"
	on:mousemove={handleMouseMove}
	on:mousedown={handleMouseDown}
	on:mouseup={handleMouseUp}
	on:mouseout={handleMouseOut}
/>

<style>
canvas {
  max-width: 100%;
  display: block;
}
</style>
