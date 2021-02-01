<script>
	import Card from "./Card.svelte";
	import { fade } from "svelte/transition";


	let sdigs = 5,
		wdigs = 5,
		zl = 3,
		zr = 3,
		b = 3,
		y = 1.2,
		yQ,
		s = 0.12,
		n = 0.02,
		g = 9.81,
		A,
		AQ,
		T,
		TQ,
		WP,
		WPQ,
		R,
		RQ,
		Q,
		QQ = 10,
		v,
		vQ,
		E,
		EQ,
		NF,
		NFQ,
		yc,
		ycQ,
		Ac,
		AcQ,
		vc,
		vcQ,
		Emin,
		EminQ,
		WPc,
		WPcQ,
		Rc,
		RcQ,
		Sc,
		ScQ;

	let specifyY = true;
	let valid = true;

	calculate();
	// calculateFromQ();

	document.addEventListener("DOMContentLoaded", katexify);
	document.addEventListener("input", katexify);
	document.addEventListener("click", katexify);
	document.addEventListener("change", katexify);

	function calculate() {
		if (s < 0) {
			s = Math.abs(s);
		} else if (s == 0) {
			valid = false;
		} else {
			valid = true;
		}

		n = n <= 0 ? 0.01 : n;
		g = g < 9.8 ? 9.8 : g;
		b = b < 0 ? 0 : b;
		b = b == 0 && zl == 0 && zr == 0 ? 0.1 : b;
		y = y <= 0 ? 1 : y;
		QQ = QQ <= 0 ? 10 : QQ;
		if (valid) {
			calculateFromY();
			calculateFromQ();
		}
	}

	function calculateFromY() {
		A = Number(sd(getArea(y), wdigs));
		T = Number(sd(b, wdigs));
		WP = Number(sd(getWP(y), wdigs));
		R = Number(sd(getR(y), wdigs));
		Q = Number(sd(getQ(y), wdigs));
		v = Number(sd(Q / A, wdigs));
		E = Number(sd(y + v ** 2 / g / 2, wdigs));
		NF = Number(sd(getNF(v, A, T), wdigs));
		yc = Number(sd(getYc(Q), wdigs));
		Ac = Number(sd(getArea(yc), wdigs));
		vc = Number(sd(Q / Ac, wdigs));
		Emin = Number(sd(+yc + vc ** 2 / 2 / g, wdigs));
		WPc = Number(sd(getWP(yc), wdigs));
		Rc = Number(sd(getR(yc), wdigs));
		Sc = Number(sd(getCriticalSlope(vc, Rc), wdigs));
	}

	function calculateFromQ() {
		yQ = Number(sd(getY(), wdigs));
		AQ = Number(sd(getArea(yQ), wdigs));
		vQ = Number(sd(QQ / AQ, wdigs));
		EQ = Number(sd(yQ + vQ ** 2 / g / 2, wdigs));
		TQ = Number(sd(b + (zl + zr) * yQ, wdigs));
		NFQ = Number(sd(getNF(vQ, AQ, TQ), wdigs));
		ycQ = Number(sd(getYc(QQ), wdigs));
		AcQ = Number(sd(getArea(ycQ), wdigs));
		vcQ = Number(sd(QQ / AcQ, wdigs));
		EminQ = Number(sd(+ycQ + vcQ ** 2 / 2 / g, wdigs));
		WPcQ = Number(sd(getWP(ycQ), wdigs));
		RcQ = Number(sd(getR(ycQ), wdigs));
		ScQ = Number(sd(getCriticalSlope(vcQ, RcQ), wdigs));
	}



	function getArea(depth) {
		return b * depth;
	}

	function getWP(depth) {
		return b + depth * 2;
	}

	function getR(depth) {
		return getArea(depth) / getWP(depth);
	}

	function getQ(depth) {
		return (1 / n) * getArea(depth) * getR(depth) ** (2 / 3) * (s / 100) ** 0.5;
	}

	function getNF(v, A, T) {
		return v / ((g * A) / T) ** 0.5;
	}
	function getNFfromY(y, Q) {
		const T = b + (zl + zr) * y;
		const A = b * y + (y ** 2 / 2) * (zl + zr);
		// return Q / A / ((g * A) / T) ** 0.5;
		return (Q * (T / g) ** 0.5) / A ** 1.5;
	}

	function getCriticalSlope(vc, Rc) {
		return ((n * vc) / Rc ** (2 / 3)) ** 2 * 100;
	}

	function getY(underY = 0, overY = 42) {
		let delta = 1 / 10 ** (wdigs + 1),
			increment = 0.05,
			mid = (underY + overY) / 2;

		if (Math.abs(underY - overY) < delta) {
			return (underY + overY) * 0.5;
		}

		while (getQ(underY + increment) < QQ) {
			underY += increment;
			overY = underY + increment;
		}

		// search;
		if (getQ(mid) < QQ) {
			return getY(mid, overY);
		} else {
			return getY(underY, mid);
		}
	}

	function getYc(Q, shallow = 0, deep = 100) {
		return (Q*Q/g/b/b)**(1/3);
	}

	function sd(num, sigDigs = sdigs) {
		return parseFloat(num).toPrecision(sigDigs);
	}

	katexify();
</script>

<main>
	<section class="intro">
		This calculator is for normal (uniform) flow in an open channel with a
		rectangular cross-section.
		<p />
		Clicking on a downward triangle will display the associated calculations for
		each result.
		<p />
		Either flow depth, !$y!$, in metres (!$\mathsf&lbrace;m&rbrace;!$) or volume
		flow, !$Q!$, in cubic metres per second (!$\mathsf&lbrace;m^3\!/s&rbrace;!$)
		must be specified. But not both. If !$y!$ is specified, then the corresponding
		!$Q!$ will be calculated. If !$Q!$ is specified, then the corresponding flow
		depth !$y!$ will be calculated.
		<p />

		<label class="yQlayout">
			<input type="radio" bind:group={specifyY} value={true} />
			<span class="radio__label">Specify !$y!$</span>
		</label>

		<label class="yQlayout">
			<input type="radio" bind:group={specifyY} value={false} />
			<span>Specify !$Q!$</span>
		</label>
	</section>

	<section class="fig width75">
		<img
			src="./images/rectangularChannelSectionQ.png"
			alt="rectangular channel section"
		/>
		{#if specifyY}
			<img
				class="super"
				src="./images/rectangularChannelSectionY.png"
				alt="rectangular channel section by depth"
				in:fade={{ duration: 750 }}
				out:fade={{ duration: 500 }}
			/>
		{/if}

		<form on:submit|preventDefault={calculate}>
			<label class="b">
				!$b=!$
				<input
					type="number"
					step="any"
					required
					bind:value={b}
					on:input={calculate}
				/>
				m
			</label>
			<!-- <label class="zl">
				!$z_L=!$
				<input
					type="number"
					step="any"
					required
					bind:value={zl}
					on:input={calculate}
				/>
			</label> -->
			<!-- <label class="zr">
				!$z_R=!$
				<input
					type="number"
					step="0.000001"
					required
					bind:value={zr}
					on:input={calculate}
				/>
			</label> -->
			{#if specifyY}
				<label class="y">
					!$y=!$
					<input
						type="number"
						step="0.000001"
						required
						bind:value={y}
						on:input={calculate}
					/>
					!$\mathsf&lbrace;m&rbrace;!$
				</label>
			{:else}
				<label class="Q">
					!$Q=!$
					<input
						type="number"
						step="0.000001"
						required
						bind:value={QQ}
						on:input={calculate}
					/>
					!$\mathsf&lbrace;m^3\!/s&rbrace;!$
				</label>
			{/if}

			<label class="s">
				Channel slope, !$S=!$
				<input
					type="number"
					step="0.000001"
					required
					bind:value={s}
					on:input={calculate}
				/>
				!$\small \% !$
			</label>
			<label class="n">
				Manning's !$n=!$
				<input
					type="number"
					step="0.000001"
					required
					bind:value={n}
					on:input={calculate}
				/>
				!$\small \mathsf &lbrace; s/m^&lbrace;1/3&rbrace; &rbrace; !$
			</label>
			<label class="g">
				!$g=!$
				<input
					type="number"
					step="0.000001"
					required
					bind:value={g}
					on:input={calculate}
				/>
				!$\small \mathsf &lbrace; m/s^2 &rbrace; !$
			</label>
		</form>
	</section>

	<section class="results">
		{#if specifyY && valid}
			<section class="normal">
				<h1>Normal (Uniform) Flow</h1>
				<Card
					answer="Flow Area: !$ A = {A} \mathsf &lcub; \,m^2 &rcub; !$"
					solution="$$
						\begin&lcub;aligned&rcub;
							A &= b\cdot y \\
							&= {b}\cdot {y}\\
							&= {A} \,\mathsf&lbrace;m^2&rbrace;
						\end&lbrace;aligned&rbrace; 
					$$"
				/>

				<Card
					answer="Wetted Perimeter: !$ W\!P = {WP}\, \mathsf m !$"
					solution="$$
					\begin&lbrace;aligned&rbrace;
						WP &= b + 2\cdot y \\
						&= {b}+2\cdot{y} \\
						&= {WP} \,\mathsf&lbrace;m&rbrace;
					\end&lbrace;aligned&rbrace;
				$$"
				/>

				<Card
					answer="Hydraulic Radius: !$ R = {R}\, \mathsf m !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							R &= A/W\!P \\
							&= {A}/{WP} \\
							&= {R} \,\mathsf&lbrace;m&rbrace;
						\end&lbrace;aligned&rbrace;
					$$"
				/>

				<Card
					answer="Average Flow Velocity: !$ v = {v}\, \mathsf &lbrace; m/s &rbrace; !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							v &= \frac 1n R^&lbrace 2/3 &rbrace;S^&lbrace 1/2 &rbrace; \\
							&= \frac 1&lbrace; {n}&rbrace; ({R})^&lbrace 2/3 &rbrace;({Number(
						sd(s / 100, sdigs)
					)})^&lbrace; 1/2 &rbrace; \\
							&= {v} \,\mathsf&lbrace;m/s&rbrace;
						\end&lbrace;aligned&rbrace;
					$$"
				/>

				<Card
					answer="Flow Rate: !$ Q = {Q}\,\mathsf &lbrace; m^3\!/s &rbrace !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							Q &= Av \\
							&= ({A})\cdot({v}) \\
							&= {Q} \,\mathsf&lbrace;m^3\!/s&rbrace;
						\end&lbrace;aligned&rbrace;
					$$"
				/>

				<Card
					answer="Specific Energy: !$ E = {E} \,\mathsf&lbrace;m&rbrace; !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							E &= y+\frac &lbrace; v^2 &rbrace;&lbrace; 2g &rbrace; \\
							&= {y}+\frac &lbrace; {v}^2 &rbrace;&lbrace; 2({g}) &rbrace; \\
							&= {E}\,\mathsf&lbrace;m&rbrace;
						\end&lbrace;aligned&rbrace;
					$$"
				/>

				<Card
					answer="Free Surface: !$ T = {T}\, \mathsf &lbrace; m &rbrace; !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							T &= b\\
							
							&= {T}\,\mathsf&lbrace;m&rbrace;
						\end&lbrace;aligned&rbrace; 
					$$"
				/>

				<Card
					answer="Froude Number: !$N_F = {NF} !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							N_F &=  \frac&lbrace;v&rbrace;&lbrace;\sqrt&lbrace;g(A/T)&rbrace;&rbrace; \\
								&=  \frac&lbrace;{v}&rbrace;&lbrace;\sqrt&lbrace;{g}({A}/{T})&rbrace;&rbrace; \\
								&= {NF}
						\end&lbrace;aligned&rbrace;
					$$"
				/>
			</section>
		{:else if specifyY && !valid}
			<section class="message">
				For a slope of 0, no flow can be calculated. And, without a flow
				specified, critical conditions are not defined.
			</section>
		{/if}
		{#if specifyY && valid}
			<section class="critical">
				<h1>Critical Flow</h1>
				<Card
					answer="For !$Q={Q}\,\mathsf&lbrace;m^3\!/s&rbrace;!$, Critical Depth: !$y_c = {yc} \,\mathsf&lbrace;m&rbrace; !$"
					solution="$$
					\begin&lbrace;aligned&rbrace;
						N_F &= 1 \\
						\Rightarrow v_c &= \sqrt&lbrace; g(A_c/T_c)&rbrace; \\
						\Rightarrow \left(\frac&lbrace;Q&rbrace;&lbrace;A_c&rbrace;\right)^2 &= g(A_c/T_c) \\
                        \Rightarrow \frac&lbrace;Q^2&rbrace;&lbrace;g&rbrace; &= \frac&lbrace;A_c^3&rbrace;&lbrace;T_c&rbrace; \\
                        &= \frac&lbrace;\left(b\cdot y_c\right)^3&rbrace;&lbrace;b&rbrace;	\\
                        \Rightarrow y_c^3 &=	 \frac&lbrace; b\cdot Q^2 &rbrace;&lbrace;b^3\cdot g&rbrace;\\				
                        \Rightarrow y_c &=	 \left(\frac&lbrace; Q^2 &rbrace;&lbrace;b^2\cdot g&rbrace;\right)^&lbrace;\!1/3&rbrace;\\				
                         &=	 \left(\frac&lbrace; {Q}^2 &rbrace;&lbrace;{b}^2\cdot {g}&rbrace;\right)^&lbrace;\!1/3&rbrace;\\				
					\end&lbrace;aligned&rbrace;$$
				
					
					$$ y_c = {yc} \,\mathsf&lbrace;m&rbrace; $$        
				"
				/>

				<Card
					answer="Critical Velocity, !$v_c = {vc}  \,\mathsf&lbrace;m/s&rbrace; !$"
					solution="$$
					\begin&lbrace;aligned&rbrace;
						A_c &= b\cdot y_c \\
						&= {b}\cdot {yc} \\
						&= {Ac} \,\mathsf&lbrace;m^2&rbrace;\\\\
						v_c &= Q/A_c \\
						&= {Q}/{Ac} \\
						&= {vc} \,\mathsf&lbrace;m/s&rbrace;
					\end&lbrace;aligned&rbrace;
				$$"
				/>

				<Card
					answer="Minimum Specific Energy, !$ E_&lbrace;min&rbrace; = {Emin}\,\mathsf&lbrace;m&rbrace;!$"
					solution="$$
					\begin&lbrace;aligned&rbrace;
						E_&lbrace;min&rbrace; &= y_c+\frac &lbrace; v_c^2 &rbrace;&lbrace; 2g &rbrace; \\
						&= {yc}+\frac &lbrace; {vc}^2 &rbrace;&lbrace; 2({g}) &rbrace; \\
						&= {Emin} \,\mathsf&lbrace;m&rbrace;
					\end&lbrace;aligned&rbrace;
				$$"
				/>

				<Card
					answer="Slope for Critical Flow, !$ S_c = {Sc}\,\mathsf&lbrace;\%&rbrace; !$"
					solution="$$
					\begin&lbrace;aligned&rbrace;
                        A_c &= b\cdot y_c \\
                        &= {b}\cdot {yc} \\
						&= {Ac} \,\mathsf&lbrace;m^2&rbrace;
						
                        \\\\    
                        
						WP &= b + 2\cdot y_c \\
						&= {b}+2\cdot{yc} \\
						&= {WPc} \,\mathsf&lbrace;m&rbrace;\\\\
						
						

						R_c &= A_c/W\!P_c \\
						&= {Ac}/{WPc} \\
						&= {Rc} \,\mathsf&lbrace;m&rbrace;\\\\
						v_c &= \frac 1n R_c^&lbrace 2/3 &rbrace;S_c^&lbrace 1/2 &rbrace; \\
						\Rightarrow S_c &= \left(\frac &lbrace; nv_c &rbrace;&lbrace; R_c^&lbrace;2/3&rbrace; &rbrace;\right)^2 \\
						&= \left(\frac &lbrace; {n}\times{vc} &rbrace;&lbrace; {Rc}^&lbrace;2/3&rbrace; &rbrace;\right)^2\\
						&= {Number(sd(						
						Sc/100,wdigs))} \\
						&= {Sc}\% \\
					\end&lbrace;aligned&rbrace;
				$$"
				/>
			</section>
		{/if}
		{#if !specifyY && valid}
			<section class="normal">
				<h1>Normal (Uniform) Flow</h1>
				<Card
					answer="Depth of flow: !$ y = {yQ} \mathsf &lcub; \,m &rcub; !$"
					solution="$$
					\begin&lcub;aligned&rcub;
						Q &= \frac 1n AR^&lbrace; 2/3 &rbrace;S^&lbrace; 1/2 &rbrace; \\
							&= \frac 1n A(A/W\!P)^&lbrace; 2/3 &rbrace;S^&lbrace; 1/2 &rbrace; \\
							&= \frac 1n \cdot \frac&lbrace;A^&lbrace;5/3&rbrace;&rbrace;&lbrace;W\!P^&lbrace;2/3&rbrace;&rbrace;\cdot S^&lbrace; 1/2 &rbrace; \\
							&= \frac 1n \cdot \frac&lbrace;\left(\left(b+\left( z_L+z_R \right)\!/2\cdot y \right) \cdot y\right)^&lbrace;5/3&rbrace;&rbrace;&lbrace;\left(b+\left(\sqrt &lbrace; 1+z_L^2 &rbrace; + \sqrt &lbrace; 1+z_R^2 &rbrace;\right)\cdot y\right)^&lbrace;2/3&rbrace;&rbrace;\cdot S^&lbrace; 1/2 &rbrace; \\
						\Rightarrow {QQ} &= \frac 1&lcub;{n}&rcub; \cdot \frac&lbrace;\left(\left({b}+\left( {zl}+{zr} \right)\!/2\cdot y \right) \cdot y\right)^&lbrace;5/3&rbrace;&rbrace;&lbrace;\left({b}+\left(\sqrt &lbrace; 1+{zl}^2 &rbrace; + \sqrt &lbrace; 1+{zr}^2 &rbrace;\right)\cdot y\right)^&lbrace;2/3&rbrace;&rbrace;\cdot\left( {s /
						100}\right)^&lbrace; 1/2 &rbrace; 		
					\end&lbrace;aligned&rbrace; 
					$$
					This equation has a single unknown, !$y!$. But it is complicated and it isn't possible to solve this equation analytically for !$y!$. 'Trial and error' or 'Guess and check' methods can be used to narrow in on the solution but it is usually simpler, and sufficient, to solve it without further simplification using a numerical solver on a calculator or in a spreadsheet program(me). 
					$$ y = {yQ} \,\mathsf&lbrace;m&rbrace; $$
					"
				/>

				<Card
					answer="Flow Area: !$ A = {AQ} \mathsf &lcub; \,m^2 &rcub; !$"
					solution="$$
						\begin&lcub;aligned&rcub;
							A &= \left(b+&lcub;\left(\frac &lcub; z_L+z_R &rcub; &lcub; 2 &rcub; \right)&rcub;\cdot y \right) 		\cdot y \\
							&= \left({b}+&lcub;\left(\frac &lcub; {zl}+{zr} &rcub; &lcub; 2 &rcub; \right)&rcub;\cdot {yQ} 			\right) \cdot {yQ} \\
							&= {AQ} \,\mathsf&lbrace;m^2&rbrace;
						\end&lbrace;aligned&rbrace; 
					$$"
				/>

				<Card
					answer="Average Flow Velocity: !$ v = {vQ}\, \mathsf &lbrace; m/s &rbrace; !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							v &= Q/A \\
							v &= {QQ}/{AQ} \\					
							&= {vQ} \,\mathsf&lbrace;m/s&rbrace;
						\end&lbrace;aligned&rbrace;
					$$"
				/>

				<Card
					answer="Specific Energy: !$ E = {EQ} \,\mathsf&lbrace;m&rbrace; !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							E &= y+\frac &lbrace; v^2 &rbrace;&lbrace; 2g &rbrace; \\
							&= {yQ}+\frac &lbrace; {vQ}^2 &rbrace;&lbrace; 2({g}) &rbrace; \\
							&= {EQ}\,\mathsf&lbrace;m&rbrace;
						\end&lbrace;aligned&rbrace;
					$$"
				/>

				<Card
					answer="Free Surface: !$ T = {TQ}\, \mathsf &lbrace; m &rbrace; !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							T &= b+(z_L+z_R)\cdot y \\
							&= {b}+({zl}+{zr})\cdot {yQ} \\
							&= {TQ}\,\mathsf&lbrace;m&rbrace;
						\end&lbrace;aligned&rbrace; 
				$$"
				/>

				<Card
					answer="Froude Number: !$N_F = {NFQ} !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							N_F &=  \frac&lbrace;v&rbrace;&lbrace;\sqrt&lbrace;g(A/T)&rbrace;&rbrace; \\
								&=  \frac&lbrace;{vQ}&rbrace;&lbrace;\sqrt&lbrace;{g}({AQ}/{TQ})&rbrace;&rbrace; \\
								&= {NFQ}
						\end&lbrace;aligned&rbrace;
					$$"
				/>
			</section>
		{:else if !specifyY && !valid}
			<section class="message">
				For a slope of 0, no depth can be calculated. But, for non-zero !$Q!$,
				critical flow conditions can be determined.
			</section>
		{/if}
		{#if !specifyY}
			<section class="critical">
				<h1>Critical Flow</h1>
				<Card
					answer="For !$Q={QQ}\,\mathsf&lbrace;m^3\!/s&rbrace;!$, Critical Depth: !$y_c = {ycQ} \,\mathsf&lbrace;m&rbrace; !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							N_F &= 1 \\
							\Rightarrow v_c &= \sqrt&lbrace; g(A_c/T_c)&rbrace; \\
							\Rightarrow \left(\frac&lbrace;Q&rbrace;&lbrace;A_c&rbrace;\right)^2 &= g(A_c/T_c) \\
							\Rightarrow \frac&lbrace;Q^2&rbrace;&lbrace;g&rbrace; &= \frac&lbrace;A_c^3&rbrace;&lbrace;T_c&rbrace; \\
							&= \frac&lbrace;\left(\left(b+&lcub;\left( z_L+z_R \right)&rcub;\cdot y_c/2 \right) \cdot y_c\right)^3&rbrace;&lbrace;b+(z_L+z_R)\cdot y_c&rbrace; \\
							\Rightarrow \frac&lbrace;{QQ}^2&rbrace;&lbrace;{g}&rbrace; &= \frac&lbrace;\left(\left({b}+&lcub;\left( {zl}+{zr} \right)&rcub;\cdot y_c/2 \right) \cdot y_c\right)^3&rbrace;&lbrace;{b}+({zl}+{zr})\cdot y_c&rbrace; \\
						\end&lbrace;aligned&rbrace;$$
						
						The expression above cannot be solved directly. It may be solved using trial and error methods but it is generally more convenient, and adequate, to solve it without further simplification using a numerical solver on a calculator or in a spreadsheet program(me).
						
						$$ y_c = {ycQ} \,\mathsf&lbrace;m&rbrace; $$ 

						"
				/>

				<Card
					answer="Critical Velocity, !$v_c = {vcQ}  \,\mathsf&lbrace;m/s&rbrace; !$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							A_c &= \left(b+(z_L+z_R)\cdot y_c/2\right)\cdot y_c \\
							&= \left({b}+({zl}+{zr})\cdot {ycQ}/2\right)\cdot {ycQ} \\
							&= {AcQ} \,\mathsf&lbrace;m^2&rbrace;\\\\
							v_c &= Q/A_c \\
							&= {QQ}/{AcQ} \\
							&= {vcQ} \,\mathsf&lbrace;m/s&rbrace;
						\end&lbrace;aligned&rbrace;
					$$"
				/>

				<Card
					answer="Minimum Specific Energy, !$ E_&lbrace;min&rbrace; = {EminQ}\,\mathsf&lbrace;m&rbrace;!$"
					solution="$$
						\begin&lbrace;aligned&rbrace;
							E_&lbrace;min&rbrace; &= y_c+\frac &lbrace; v_c^2 &rbrace;&lbrace; 2g &rbrace; \\
							&= {ycQ}+\frac &lbrace; {vcQ}^2 &rbrace;&lbrace; 2({g}) &rbrace; \\
							&= {EminQ} \,\mathsf&lbrace;m&rbrace;
						\end&lbrace;aligned&rbrace;
					$$"
				/>

				<Card
					answer="Slope for Critical Flow, !$ S_c = {ScQ}\,\mathsf&lbrace;\%&rbrace; !$"
					solution="$$
					\begin&lbrace;aligned&rbrace;
						A_c &= \left(b+&lcub;\left( z_L+z_R \right)&rcub;\cdot y_c/2 \right) \cdot y_c \\
						&= \left({b}+&lcub;\left( {zl}+{zr} \right)&rcub;\cdot {ycQ}/2 \right) \cdot {ycQ} \\
						&= {AcQ} \,\mathsf&lbrace;m^2&rbrace;\\\\

						W\!P_c &= b+\left(\sqrt &lbrace; 1+z_L^2 &rbrace; + \sqrt &lbrace; 1+z_R^2 &rbrace;\right)\cdot y_c \\
						&= {b}+\left(\sqrt &lbrace; 1+{zl}^2 &rbrace; + \sqrt &lbrace; 1+{zr}^2 &rbrace;\right)\cdot {ycQ} \\
						&= {WPcQ} \,\mathsf&lbrace;m&rbrace;\\\\					

						R_c &= A_c/W\!P_c \\
						&= {AcQ}/{WPcQ} \\
						&= {RcQ} \,\mathsf&lbrace;m&rbrace;\\\\
						v_c &= \frac 1n R_c^&lbrace 2/3 &rbrace;S_c^&lbrace 1/2 &rbrace; \\
						\Rightarrow S_c &= \left(\frac &lbrace; nv_c &rbrace;&lbrace; R_c^&lbrace;2/3&rbrace; &rbrace;\right)^2 \\
						&= \left(\frac &lbrace; {n}\times{vcQ} &rbrace;&lbrace; {RcQ}^&lbrace;2/3&rbrace; &rbrace;\right)^2\\
						&= {(1 /
						100) *
						ScQ} \\
						&= {ScQ}\% \\
					\end&lbrace;aligned&rbrace;
				$$"
				/>
			</section>
		{/if}
	</section>
</main>

<style>
	/* Chrome, Safari, Edge, Opera */
	input::-webkit-outer-spin-button,
	input::-webkit-inner-spin-button {
		-webkit-appearance: none;
		margin: 0;
	}
	/* Firefox */
	input[type="number"] {
		-moz-appearance: textfield;
	}
</style>
