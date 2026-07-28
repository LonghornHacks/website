<script lang="ts">
	// Ported verbatim from the Claude Design handoff ("Longhorn Hacks.dc.html").
	// The handoff shipped its behaviour as a `class Component extends DCLogic`
	// evaluated by the design tool's support.js runtime; this is the same logic
	// expressed with Svelte's lifecycle. Nothing about the markup, styling or
	// timing has been altered.
	import { onMount } from 'svelte';
	import './design.css';
	import lhhLogo from '$lib/assets/lhh-logo.png';

	// Defaults from the handoff's data-props block.
	const motion: boolean = true;
	const magnetic: boolean = true;
	const nodeDensity: number = 34;

	// handoff: `this.reduced = this.props.motion === false`
	const reduced = !motion;

	let submitLabel = $state('Send message');
	let submitTimer: ReturnType<typeof setTimeout> | undefined;

	function onSubmit(e: Event) {
		e.preventDefault();
		submitLabel = "Thanks — we'll be in touch";
		clearTimeout(submitTimer);
		submitTimer = setTimeout(() => {
			submitLabel = 'Send message';
		}, 3200);
	}

	function onTop() {
		window.scrollTo({ top: 0, behavior: reduced ? 'auto' : 'smooth' });
	}

	onMount(() => {
		const cleanups: Array<() => void> = [];
		let raf = 0;
		let countObs: IntersectionObserver | undefined;
		let revealCheck: (() => void) | undefined;
		let growApply: (() => void) | undefined;
		let moveIndicator: (() => void) | undefined;
		let hideMenu: (() => void) | undefined;

		function on(target: EventTarget, type: string, fn: EventListenerOrEventListenerObject, opts?: AddEventListenerOptions) {
			target.addEventListener(type, fn, opts);
			cleanups.push(() => target.removeEventListener(type, fn, opts));
		}

		function setupReveals() {
			const els = Array.from(document.querySelectorAll<HTMLElement>('[data-reveal]'));
			if (reduced || !els.length) return;
			const ease = 'cubic-bezier(.16,.84,.32,1)';
			els.forEach((el) => {
				el.style.opacity = '0';
				const kind = el.getAttribute('data-reveal');
				if (kind === 'mask') {
					el.style.clipPath = 'inset(0 0 100% 0)';
					el.style.transform = 'translateY(12px)';
				} else if (kind === 'left') {
					el.style.transform = 'translateX(-52px)';
				} else if (kind === 'right') {
					el.style.transform = 'translateX(52px)';
				} else {
					el.style.transform = 'translateY(32px)';
				}
				el.style.transition = 'opacity .85s ' + ease + ', transform 1s ' + ease + ', clip-path 1.1s ' + ease;
			});
			const show = (el: HTMLElement) => {
				if (el.dataset.lhShown) return;
				el.dataset.lhShown = '1';
				const delay = parseInt(el.getAttribute('data-reveal-delay') || '0', 10);
				setTimeout(() => {
					el.style.opacity = '1';
					el.style.clipPath = 'inset(0 0 0% 0)';
					el.style.transform = 'none';
				}, delay);
			};
			let ticking = false;
			const check = () => {
				ticking = false;
				const vh = window.innerHeight;
				els.forEach((el) => {
					if (el.dataset.lhShown) return;
					if (!el.offsetParent && !el.offsetHeight) return;
					const r = el.getBoundingClientRect();
					if (r.bottom <= 0) {
						el.dataset.lhShown = '1';
						el.style.opacity = '1';
						el.style.clipPath = 'inset(0 0 0% 0)';
						el.style.transform = 'none';
						return;
					}
					if (r.top < vh * 0.92) show(el);
				});
			};
			revealCheck = () => {
				if (!ticking) {
					ticking = true;
					requestAnimationFrame(check);
				}
			};
			on(window, 'scroll', revealCheck, { passive: true });
			on(window, 'resize', revealCheck);
			requestAnimationFrame(check);
			setTimeout(check, 350);
			setTimeout(check, 1200);
		}

		function setupGrow() {
			const els = Array.from(document.querySelectorAll<HTMLElement>('[data-grow]'));
			if (!els.length) return;
			if (reduced) {
				els.forEach((el) => {
					el.style.transform = 'none';
				});
				return;
			}
			let ticking = false;
			const apply = () => {
				ticking = false;
				const vh = window.innerHeight;
				els.forEach((el) => {
					const scope = el.parentElement;
					if (!scope) return;
					const r = scope.getBoundingClientRect();
					if (!r.height) return;
					const p = Math.max(0, Math.min(1, (vh * 0.86 - r.top) / (r.height * 0.72)));
					const axis = el.getAttribute('data-grow');
					el.style.transform = axis === 'y' ? 'scaleY(' + p.toFixed(4) + ')' : 'scaleX(' + p.toFixed(4) + ')';
				});
			};
			growApply = apply;
			on(
				window,
				'scroll',
				() => {
					if (!ticking) {
						ticking = true;
						requestAnimationFrame(apply);
					}
				},
				{ passive: true }
			);
			on(window, 'resize', apply);
			apply();
		}

		function setupProgress() {
			const bar = document.getElementById('lh-progress');
			if (!bar) return;
			let ticking = false;
			const apply = () => {
				ticking = false;
				const max = Math.max(1, document.documentElement.scrollHeight - window.innerHeight);
				bar.style.width = Math.min(100, (window.scrollY / max) * 100).toFixed(2) + '%';
			};
			on(
				window,
				'scroll',
				() => {
					if (!ticking) {
						ticking = true;
						requestAnimationFrame(apply);
					}
				},
				{ passive: true }
			);
			on(window, 'resize', apply);
			apply();
		}

		function setupMagnets() {
			if (reduced || magnetic === false) return;
			const els = Array.from(document.querySelectorAll<HTMLElement>('a[style],button[style]')).filter((el) => {
				const st = el.getAttribute('style') || '';
				return st.indexOf('background:#FF6A00') >= 0 || st.indexOf('background:#F7F4EF;color:#04091A') >= 0;
			});
			els.forEach((el) => {
				el.style.transition = (el.style.transition ? el.style.transition + ',' : '') + 'translate .3s cubic-bezier(.2,.7,.2,1)';
				on(el, 'pointermove', ((e: PointerEvent) => {
					const r = el.getBoundingClientRect();
					const dx = (e.clientX - (r.left + r.width / 2)) / Math.max(1, r.width);
					const dy = (e.clientY - (r.top + r.height / 2)) / Math.max(1, r.height);
					el.style.translate = (dx * 10).toFixed(1) + 'px ' + (dy * 8).toFixed(1) + 'px';
				}) as EventListener);
				on(el, 'pointerleave', () => {
					el.style.translate = '0 0';
				});
			});
		}

		function setupTilt() {
			if (reduced) return;
			const art = document.getElementById('lh-heroart');
			if (!art) return;
			const host = art.closest('section');
			if (!host) return;
			setTimeout(() => {
				art.style.animation = 'none';
				art.style.opacity = '1';
				art.style.transition = 'transform .6s cubic-bezier(.2,.7,.2,1)';
				art.style.transform = 'perspective(1100px)';
			}, 1700);
			on(host, 'pointermove', ((e: PointerEvent) => {
				if (art.style.animation !== 'none') return;
				const r = host.getBoundingClientRect();
				const dx = (e.clientX - (r.left + r.width / 2)) / Math.max(1, r.width);
				const dy = (e.clientY - (r.top + r.height / 2)) / Math.max(1, r.height);
				art.style.transition = 'transform .18s linear';
				art.style.transform =
					'perspective(1100px) rotateY(' + (dx * 11).toFixed(2) + 'deg) rotateX(' + (-dy * 9).toFixed(2) + 'deg)';
			}) as EventListener);
			on(host, 'pointerleave', () => {
				if (art.style.animation !== 'none') return;
				art.style.transition = 'transform .7s cubic-bezier(.2,.7,.2,1)';
				art.style.transform = 'perspective(1100px)';
			});
		}

		function setupCounters() {
			const nodes = Array.from(document.querySelectorAll<HTMLElement>('[data-count]'));
			if (!nodes.length) return;
			const fmt = (n: number) => n.toLocaleString('en-US');
			if (reduced) {
				nodes.forEach((n) => {
					n.textContent = fmt(parseInt(n.getAttribute('data-count') || '0', 10));
				});
				return;
			}
			countObs = new IntersectionObserver(
				(entries) => {
					entries.forEach((entry) => {
						if (!entry.isIntersecting) return;
						const el = entry.target as HTMLElement;
						const target = parseInt(el.getAttribute('data-count') || '0', 10) || 0;
						const t0 = performance.now();
						const step = (t: number) => {
							const p = Math.min(1, (t - t0) / 1700);
							el.textContent = fmt(Math.round(target * (1 - Math.pow(1 - p, 3))));
							if (p < 1) requestAnimationFrame(step);
						};
						requestAnimationFrame(step);
						countObs?.unobserve(el);
					});
				},
				{ threshold: 0.4 }
			);
			nodes.forEach((n) => countObs?.observe(n));
		}

		function setupRouter() {
			const screens = Array.from(document.querySelectorAll<HTMLElement>('[data-screen]'));
			const links = Array.from(document.querySelectorAll<HTMLElement>('[data-nav]'));
			if (!screens.length) return;
			const routeOf = () => {
				const h = (window.location.hash || '').replace(/^#/, '');
				return screens.some((s) => s.getAttribute('data-screen') === h) ? h : '/';
			};
			const apply = (initial: boolean) => {
				const route = routeOf();
				screens.forEach((s) => {
					const active = s.getAttribute('data-screen') === route;
					s.style.display = active ? 'block' : 'none';
					if (active && !reduced && !initial) {
						s.style.opacity = '0';
						s.style.transform = 'translateY(12px)';
						s.style.transition = 'opacity .5s ease, transform .6s cubic-bezier(.16,.84,.32,1)';
						requestAnimationFrame(() => {
							s.style.opacity = '1';
							s.style.transform = 'none';
						});
					}
				});
				links.forEach((a) => {
					const isOn = a.getAttribute('data-nav') === route;
					a.style.color = isOn ? '#F7F4EF' : '#A8B0C8';
					const dot = a.querySelector<HTMLElement>('[data-dot]');
					if (dot) dot.style.opacity = isOn ? '1' : '0';
				});
				moveIndicator = () => {
					const ind = document.getElementById('lh-ind');
					const active = links.filter((a) => a.getAttribute('data-nav') === route)[0] as HTMLElement | undefined;
					if (!ind || !active || !active.offsetParent) {
						if (ind) ind.style.opacity = '0';
						return;
					}
					ind.style.opacity = '1';
					ind.style.width = active.offsetWidth + 'px';
					ind.style.transform = 'translateX(' + active.offsetLeft + 'px)';
				};
				requestAnimationFrame(moveIndicator);
				setTimeout(moveIndicator, 240);
				if (!initial) window.scrollTo({ top: 0, behavior: 'auto' });
				if (revealCheck) {
					requestAnimationFrame(revealCheck);
					setTimeout(revealCheck, 120);
				}
				if (growApply) {
					requestAnimationFrame(growApply);
					setTimeout(growApply, 120);
				}
			};
			on(window, 'hashchange', () => apply(false));
			apply(true);
		}

		function setupNav() {
			const nav = document.getElementById('lh-nav');
			if (!nav) return;
			const bar = nav.firstElementChild as HTMLElement | null;
			const onScroll = () => {
				const solid = window.scrollY > 30;
				nav.style.padding = solid ? '10px clamp(12px,2.6vw,28px)' : '16px clamp(12px,2.6vw,28px)';
				if (bar) {
					bar.style.background = solid ? 'rgba(4,9,26,.93)' : 'rgba(6,12,32,.66)';
					bar.style.borderColor = solid ? 'rgba(247,244,239,.16)' : 'rgba(247,244,239,.11)';
				}
			};
			on(window, 'scroll', onScroll, { passive: true });
			onScroll();
		}

		function setupMobileMenu() {
			const burger = document.getElementById('lh-burger');
			const panel = document.getElementById('lh-mobile');
			const close = document.getElementById('lh-close');
			if (!burger || !panel) return;
			const links = Array.from(panel.querySelectorAll<HTMLElement>('[data-mlink]'));
			const showLink = (l: HTMLElement) => {
				l.style.opacity = '1';
				l.style.transform = 'translateY(0)';
			};
			const reveal = () => {
				panel.style.opacity = '1';
				links.forEach((l, i) => {
					if (reduced) {
						l.style.transition = 'none';
						showLink(l);
						return;
					}
					l.style.transition = 'opacity .4s ease, transform .5s cubic-bezier(.16,.84,.32,1)';
					setTimeout(() => showLink(l), 50 + i * 50);
					setTimeout(() => showLink(l), 700 + i * 50);
				});
			};
			const show = () => {
				panel.style.display = 'flex';
				void panel.offsetHeight;
				reveal();
				requestAnimationFrame(reveal);
				setTimeout(reveal, 0);
				document.body.style.overflow = 'hidden';
			};
			hideMenu = () => {
				panel.style.opacity = '0';
				links.forEach((l) => {
					l.style.transition = 'none';
					l.style.opacity = '0';
					l.style.transform = 'translateY(18px)';
				});
				document.body.style.overflow = '';
				setTimeout(() => {
					panel.style.display = 'none';
				}, 340);
			};
			on(burger, 'click', show);
			if (close) on(close, 'click', () => hideMenu?.());
			links.forEach((l) => on(l, 'click', () => hideMenu?.()));
		}

		function setupResponsive() {
			const sync = () => {
				const w = window.innerWidth;
				const compact = w <= 1080;
				const burger = document.getElementById('lh-burger');
				const linksRow = document.getElementById('lh-links');
				const meta = document.getElementById('lh-navmeta');
				if (burger) burger.style.display = compact ? 'flex' : 'none';
				if (linksRow) linksRow.style.display = compact ? 'none' : 'flex';
				if (meta) meta.style.display = w <= 1220 ? 'none' : 'flex';
				if (!compact && hideMenu) {
					const panel = document.getElementById('lh-mobile');
					if (panel && panel.style.display === 'flex') hideMenu();
				}
				const one = 'minmax(0,1fr)';
				const hero = document.getElementById('lh-herogrid');
				if (hero) hero.style.gridTemplateColumns = w <= 900 ? one : 'minmax(0,1.12fr) minmax(0,.88fr)';
				const orbit = document.getElementById('lh-orbitgrid');
				if (orbit) orbit.style.gridTemplateColumns = w <= 900 ? one : one + ' ' + one;
				const ph = document.getElementById('lh-proghero');
				if (ph) ph.style.gridTemplateColumns = w <= 900 ? one : 'minmax(0,1.1fr) minmax(0,.9fr)';
				const cg = document.getElementById('lh-contactgrid');
				if (cg) cg.style.gridTemplateColumns = w <= 900 ? one : 'minmax(0,1fr) minmax(0,.95fr)';
				const bento = document.getElementById('lh-bento');
				if (bento) bento.style.gridTemplateColumns = w <= 720 ? one : 'repeat(4,minmax(0,1fr))';
				if (bento)
					Array.from(bento.children).forEach((c) => {
						(c as HTMLElement).style.gridColumn = w <= 720 ? 'span 1' : '';
					});
				Array.from(document.querySelectorAll<HTMLElement>('.lh-prow')).forEach((r, i) => {
					r.style.gridTemplateColumns =
						w <= 860 ? one : i === 1 ? 'minmax(0,1.1fr) minmax(0,.9fr)' : 'minmax(0,.9fr) minmax(0,1.1fr)';
					const media = r.querySelector<HTMLElement>("div[style*='aspect-ratio']");
					if (media) media.style.order = w <= 860 ? '-1' : '';
				});
				Array.from(document.querySelectorAll<HTMLElement>('.lh-erow')).forEach((r) => {
					r.style.gridTemplateColumns = w <= 860 ? one : 'minmax(0,.24fr) minmax(0,1fr) minmax(0,.26fr)';
				});
				Array.from(document.querySelectorAll<HTMLElement>('.lh-trow')).forEach((r) => {
					r.style.gridTemplateColumns = w <= 620 ? '36px minmax(0,1fr)' : '44px minmax(0,1fr) minmax(0,1fr)';
				});
				Array.from(document.querySelectorAll<HTMLElement>('[data-hcard]')).forEach((c) => {
					c.style.display = w <= 620 ? 'none' : '';
				});
			};
			on(window, 'resize', () => {
				sync();
				if (moveIndicator) requestAnimationFrame(moveIndicator);
			});
			sync();
		}

		function setupParallax() {
			if (reduced) return;
			const art = document.getElementById('lh-heroart');
			const paras = Array.from(document.querySelectorAll<HTMLElement>('[data-para]'));
			if (!art && !paras.length) return;
			let ticking = false;
			const apply = () => {
				ticking = false;
				if (art) art.style.translate = '0 ' + (Math.min(window.scrollY, 900) * 0.07).toFixed(1) + 'px';
				const vh = window.innerHeight;
				paras.forEach((el) => {
					const r = el.getBoundingClientRect();
					const k = parseFloat(el.getAttribute('data-para') || '') || 0.1;
					el.style.translate = '0 ' + ((vh - r.top - vh * 0.5) * k).toFixed(1) + 'px';
				});
			};
			on(
				window,
				'scroll',
				() => {
					if (!ticking) {
						ticking = true;
						requestAnimationFrame(apply);
					}
				},
				{ passive: true }
			);
			on(window, 'resize', apply);
			apply();
		}

		function setupCanvas() {
			const canvas = document.getElementById('lh-canvas') as HTMLCanvasElement | null;
			if (!canvas) return;
			const ctx = canvas.getContext('2d');
			if (!ctx) return;
			const dpr = Math.min(window.devicePixelRatio || 1, 2);
			let w = 0,
				h = 0;
			let nodes: Array<{ x: number; y: number; vx: number; vy: number; r: number; warm: boolean }> = [];
			const density = nodeDensity ?? 34;
			const paint = () => {
				ctx.clearRect(0, 0, w, h);
				for (let i = 0; i < nodes.length; i++) {
					const n = nodes[i];
					for (let j = i + 1; j < nodes.length; j++) {
						const m = nodes[j];
						const dx = n.x - m.x,
							dy = n.y - m.y,
							d2 = dx * dx + dy * dy;
						if (d2 < 32000) {
							const a = 1 - d2 / 32000;
							ctx.beginPath();
							ctx.moveTo(n.x, n.y);
							ctx.lineTo(m.x, m.y);
							ctx.strokeStyle =
								n.warm || m.warm
									? 'rgba(255,138,43,' + (a * 0.3).toFixed(3) + ')'
									: 'rgba(122,152,225,' + (a * 0.24).toFixed(3) + ')';
							ctx.lineWidth = 0.6;
							ctx.stroke();
						}
					}
				}
				for (const n of nodes) {
					ctx.beginPath();
					ctx.arc(n.x, n.y, n.r, 0, Math.PI * 2);
					ctx.fillStyle = n.warm ? 'rgba(255,138,43,.95)' : 'rgba(150,178,245,.55)';
					ctx.fill();
				}
			};
			const resize = () => {
				const r = canvas.getBoundingClientRect();
				w = r.width;
				h = r.height;
				canvas.width = Math.max(1, Math.round(w * dpr));
				canvas.height = Math.max(1, Math.round(h * dpr));
				ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
				const count = Math.max(12, Math.round(density * Math.min(1.3, w / 1200)));
				nodes = new Array(count).fill(0).map(() => ({
					x: Math.random() * w,
					y: Math.random() * h,
					vx: (Math.random() - 0.5) * 0.14,
					vy: (Math.random() - 0.5) * 0.14,
					r: Math.random() * 1.3 + 0.8,
					warm: Math.random() > 0.7
				}));
				if (reduced) paint();
			};
			resize();
			on(window, 'resize', resize);
			if (reduced) return;
			const draw = () => {
				for (const n of nodes) {
					n.x += n.vx;
					n.y += n.vy;
					if (n.x < 0 || n.x > w) n.vx *= -1;
					if (n.y < 0 || n.y > h) n.vy *= -1;
				}
				paint();
				raf = requestAnimationFrame(draw);
			};
			draw();
		}

		setupReveals();
		setupCounters();
		setupRouter();
		setupNav();
		setupMobileMenu();
		setupResponsive();
		setupCanvas();
		setupParallax();
		setupProgress();
		setupMagnets();
		setupTilt();
		setupGrow();

		return () => {
			cancelAnimationFrame(raf);
			if (countObs) countObs.disconnect();
			cleanups.forEach((fn) => fn());
			clearTimeout(submitTimer);
		};
	});
</script>

<svelte:head>
	<title>Longhorn Hacks — Computer science education for every community</title>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="" />
	<link
		href="https://fonts.googleapis.com/css2?family=Archivo:wdth,wght@75..125,400..800&family=Space+Grotesk:wght@500;600;700&family=Manrope:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div style="position:relative;background:#04091A">

<header id="lh-nav" style="position:fixed;top:0;left:0;right:0;z-index:60;padding:16px clamp(12px,2.6vw,28px);transition:padding .4s cubic-bezier(.2,.7,.2,1)">
  <div id="lh-progress" aria-hidden="true" style="position:absolute;top:0;left:0;height:2px;width:0%;background:#FF6A00;box-shadow:0 0 14px rgba(255,106,0,.9)"></div>
  <nav style="position:relative;overflow:hidden;max-width:1340px;margin:0 auto;display:flex;align-items:center;justify-content:space-between;gap:14px;padding:9px 9px 9px 15px;border-radius:999px;border:1px solid rgba(247,244,239,.11);background:rgba(6,12,32,.66);backdrop-filter:blur(22px);-webkit-backdrop-filter:blur(22px);box-shadow:0 18px 50px rgba(0,0,0,.5)">
    <span aria-hidden="true" style="position:absolute;inset:-40px;pointer-events:none;opacity:.5;background-image:radial-gradient(rgba(247,244,239,.15) 1px,transparent 1px);background-size:18px 18px;animation:lhDrift 34s linear infinite"></span>
    <span aria-hidden="true" style="position:absolute;left:0;bottom:0;width:22%;height:1.5px;background:#FF6A00;opacity:.75;animation:lhRail 9s linear infinite;pointer-events:none"></span>
    <a href="#/" style="position:relative;display:flex;align-items:center;gap:11px;color:#F7F4EF">
      <span style="display:grid;place-items:center;width:38px;height:38px;border-radius:50%;background:#F7F4EF;flex:none">
        <img src={lhhLogo} alt="Longhorn Hacks logo" width="30" height="30" style="display:block;width:30px;height:30px;object-fit:contain" />
      </span>
      <span style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:14.5px;letter-spacing:.02em;text-transform:uppercase;white-space:nowrap">Longhorn Hacks</span>
    </a>

    <div id="lh-links" style="position:relative;display:flex;align-items:center;gap:3px;padding:4px;border-radius:999px;background:rgba(247,244,239,.045);border:1px solid rgba(247,244,239,.07)">
      <span id="lh-ind" aria-hidden="true" style="position:absolute;top:4px;left:0;height:calc(100% - 8px);width:0;border-radius:999px;background:rgba(255,106,0,.18);border:1px solid rgba(255,138,43,.5);opacity:0;transition:transform .45s cubic-bezier(.2,.7,.2,1),width .45s cubic-bezier(.2,.7,.2,1),opacity .3s ease;pointer-events:none"></span>
      <a data-nav="/" href="#/" style="position:relative;display:flex;align-items:center;gap:7px;padding:9px 16px;border-radius:999px;font-family:'JetBrains Mono',monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;color:#A8B0C8;transition:color .3s ease,background .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp0"><span data-dot="1" style="width:5px;height:5px;border-radius:50%;background:#FF6A00;opacity:0;transition:opacity .3s ease"></span>Home</a>
      <a data-nav="/about" href="#/about" style="position:relative;display:flex;align-items:center;gap:7px;padding:9px 16px;border-radius:999px;font-family:'JetBrains Mono',monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;color:#A8B0C8;transition:color .3s ease,background .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp0"><span data-dot="1" style="width:5px;height:5px;border-radius:50%;background:#FF6A00;opacity:0;transition:opacity .3s ease"></span>About</a>
      <a data-nav="/programs" href="#/programs" style="position:relative;display:flex;align-items:center;gap:7px;padding:9px 16px;border-radius:999px;font-family:'JetBrains Mono',monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;color:#A8B0C8;transition:color .3s ease,background .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp0"><span data-dot="1" style="width:5px;height:5px;border-radius:50%;background:#FF6A00;opacity:0;transition:opacity .3s ease"></span>Programs</a>
      <a data-nav="/events" href="#/events" style="position:relative;display:flex;align-items:center;gap:7px;padding:9px 16px;border-radius:999px;font-family:'JetBrains Mono',monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;color:#A8B0C8;transition:color .3s ease,background .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp0"><span data-dot="1" style="width:5px;height:5px;border-radius:50%;background:#FF6A00;opacity:0;transition:opacity .3s ease"></span>Events</a>
      <a data-nav="/contact" href="#/contact" style="position:relative;display:flex;align-items:center;gap:7px;padding:9px 16px;border-radius:999px;font-family:'JetBrains Mono',monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;color:#A8B0C8;transition:color .3s ease,background .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp0"><span data-dot="1" style="width:5px;height:5px;border-radius:50%;background:#FF6A00;opacity:0;transition:opacity .3s ease"></span>Contact</a>
    </div>

    <div style="position:relative;display:flex;align-items:center;gap:8px">
      <span id="lh-navmeta" style="display:flex;align-items:center;gap:7px;padding:0 8px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;color:#7D87A5;white-space:nowrap"><span style="width:5px;height:5px;border-radius:50%;background:#FF6A00;animation:lhNode 2.2s ease-in-out infinite"></span>501(C)(3)</span>
      <a href="#/contact" style="display:inline-flex;align-items:center;gap:9px;padding:11px 20px;border-radius:999px;background:#FF6A00;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;font-weight:500;box-shadow:0 8px 26px rgba(255,106,0,.4);transition:background .3s ease,box-shadow .3s ease,transform .3s cubic-bezier(.2,.7,.2,1);white-space:nowrap" class="scp1">Donate</a>
      <button id="lh-burger" type="button" aria-label="Open menu" style="display:none;width:40px;height:40px;border-radius:50%;border:1px solid rgba(247,244,239,.18);background:transparent;color:#F7F4EF;align-items:center;justify-content:center;flex-direction:column;gap:4px;cursor:pointer;flex:none">
        <span style="display:block;width:15px;height:1.5px;background:#F7F4EF"></span>
        <span style="display:block;width:15px;height:1.5px;background:#F7F4EF"></span>
      </button>
    </div>
  </nav>
</header>

<div id="lh-mobile" style="position:fixed;inset:0;z-index:70;background:#04091A;display:none;flex-direction:column;padding:20px clamp(18px,6vw,44px) 36px;opacity:0;transition:opacity .35s ease">
  <div style="display:flex;justify-content:space-between;align-items:center;height:48px">
    <span style="font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.24em;color:#7D87A5">MENU</span>
    <button id="lh-close" type="button" aria-label="Close menu" style="width:42px;height:42px;border-radius:50%;border:1px solid rgba(247,244,239,.18);background:transparent;color:#F7F4EF;font-size:20px;cursor:pointer">×</button>
  </div>
  <div style="display:flex;flex-direction:column;margin-top:auto;margin-bottom:auto">
    <a data-mlink="1" href="#/" style="display:flex;align-items:baseline;gap:16px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(30px,8vw,46px);letter-spacing:-.01em;text-transform:uppercase;color:#F7F4EF;opacity:0;transform:translateY(18px)"><span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:#FF6A00">01</span>Home</a>
    <a data-mlink="1" href="#/about" style="display:flex;align-items:baseline;gap:16px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(30px,8vw,46px);letter-spacing:-.01em;text-transform:uppercase;color:#F7F4EF;opacity:0;transform:translateY(18px)"><span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:#FF6A00">02</span>About</a>
    <a data-mlink="1" href="#/programs" style="display:flex;align-items:baseline;gap:16px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(30px,8vw,46px);letter-spacing:-.01em;text-transform:uppercase;color:#F7F4EF;opacity:0;transform:translateY(18px)"><span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:#FF6A00">03</span>Programs</a>
    <a data-mlink="1" href="#/events" style="display:flex;align-items:baseline;gap:16px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(30px,8vw,46px);letter-spacing:-.01em;text-transform:uppercase;color:#F7F4EF;opacity:0;transform:translateY(18px)"><span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:#FF6A00">04</span>Events</a>
    <a data-mlink="1" href="#/contact" style="display:flex;align-items:baseline;gap:16px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);border-bottom:1px solid rgba(247,244,239,.12);font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(30px,8vw,46px);letter-spacing:-.01em;text-transform:uppercase;color:#F7F4EF;opacity:0;transform:translateY(18px)"><span style="font-family:'JetBrains Mono',monospace;font-size:11px;color:#FF6A00">05</span>Contact</a>
  </div>
  <a data-mlink="1" href="#/contact" style="text-align:center;padding:17px;border-radius:999px;background:#FF6A00;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:12px;letter-spacing:.18em;text-transform:uppercase;opacity:0;transform:translateY(18px)">Donate</a>
</div>

<main id="lh-main" style="position:relative">

<div data-screen="/" data-screen-label="Home">

  <section style="position:relative;overflow:hidden;background:#04091A;padding:clamp(126px,16vh,180px) clamp(18px,3.4vw,40px) clamp(40px,5vw,64px);min-height:100vh;display:flex;align-items:center">
    <div aria-hidden="true" style="position:absolute;right:-16%;bottom:-38%;width:min(900px,88vw);aspect-ratio:1;pointer-events:none;border-radius:50%;border:1px solid rgba(255,138,43,.3);display:grid;place-items:center">
      <div style="width:74%;aspect-ratio:1;border-radius:50%;border:1px solid rgba(255,138,43,.2);display:grid;place-items:center">
        <div style="width:62%;aspect-ratio:1;border-radius:50%;border:1px dashed rgba(255,138,43,.16)"></div>
      </div>
    </div>
    <div aria-hidden="true" style="position:absolute;left:0;bottom:clamp(30px,7vw,88px);width:clamp(90px,14vw,230px);height:6px;background:#FF6A00;pointer-events:none"></div>
    <div aria-hidden="true" style="position:absolute;left:0;right:0;bottom:0;height:1px;background:rgba(247,244,239,.12);pointer-events:none"></div>
    <div style="position:absolute;inset:0;pointer-events:none;opacity:.5;background-image:radial-gradient(rgba(247,244,239,.16) 1px,transparent 1px);background-size:26px 26px;mask-image:radial-gradient(70% 70% at 40% 40%,#000,transparent 76%);-webkit-mask-image:radial-gradient(70% 70% at 40% 40%,#000,transparent 76%)"></div>
    <canvas id="lh-canvas" style="position:absolute;inset:0;width:100%;height:100%;display:block;opacity:.7;pointer-events:none"></canvas>

    <div style="position:relative;z-index:2;max-width:1340px;margin:0 auto;width:100%">
      <div id="lh-herogrid" style="display:grid;grid-template-columns:minmax(0,1.12fr) minmax(0,.88fr);gap:clamp(28px,4vw,56px);align-items:center">
        <div>
          <div style="display:inline-flex;align-items:center;gap:10px;padding:7px 14px 7px 10px;border-radius:999px;border:1px solid rgba(255,138,43,.34);background:rgba(255,106,0,.09);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.18em;color:#FFA65C;opacity:0;animation:lhRise .8s .1s cubic-bezier(.16,.84,.32,1) forwards">
            <span style="width:6px;height:6px;border-radius:50%;background:#FF6A00;box-shadow:0 0 12px #FF6A00;animation:lhNode 2.2s ease-in-out infinite"></span>AUSTIN, TEXAS — NONPROFIT
          </div>
          <h1 style="margin:clamp(20px,2.6vw,32px) 0 0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 118;font-weight:700;font-size:clamp(38px,6.6vw,96px);line-height:.9;letter-spacing:-.02em;text-transform:uppercase;opacity:0;animation:lhRise .95s .22s cubic-bezier(.16,.84,.32,1) forwards">
            <span style="display:block;color:#F7F4EF">Bridging the</span>
            <span style="display:block;color:transparent;-webkit-text-stroke:1.6px #FF6A00">Digital Divide</span>
          </h1>
          <p style="margin:clamp(20px,2.4vw,30px) 0 0;max-width:44ch;font-size:clamp(15.5px,1.3vw,18px);line-height:1.62;color:#B9C0D4;opacity:0;animation:lhRise .95s .36s cubic-bezier(.16,.84,.32,1) forwards">Free computer science and AI education — plus the hardware to use it — for communities across Texas the tech economy left out. One line of code at a time.</p>
          <div style="display:flex;flex-wrap:wrap;gap:12px;margin-top:clamp(26px,3vw,38px);opacity:0;animation:lhRise .95s .48s cubic-bezier(.16,.84,.32,1) forwards">
            <a href="#/contact" style="display:inline-flex;align-items:center;gap:12px;padding:8px 8px 8px 22px;border-radius:999px;background:#FF6A00;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.14em;text-transform:uppercase;box-shadow:0 12px 38px rgba(255,106,0,.42);transition:background .3s ease,box-shadow .3s ease,transform .3s cubic-bezier(.2,.7,.2,1)" class="scp2">Get involved<span style="display:grid;place-items:center;width:32px;height:32px;border-radius:50%;background:#04091A;color:#FF8A2B;font-size:13px">→</span></a>
            <a href="#/programs" style="display:inline-flex;align-items:center;gap:12px;padding:8px 8px 8px 22px;border-radius:999px;border:1px solid rgba(247,244,239,.2);color:#F7F4EF;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.14em;text-transform:uppercase;transition:border-color .3s ease,color .3s ease,transform .3s cubic-bezier(.2,.7,.2,1)" class="scp3">Explore programs<span style="display:grid;place-items:center;width:32px;height:32px;border-radius:50%;background:rgba(247,244,239,.08);font-size:13px">→</span></a>
          </div>
        </div>

        <div id="lh-heroart" style="position:relative;display:grid;place-items:center;min-height:min(58vw,440px);opacity:0;animation:lhRise 1.1s .4s cubic-bezier(.16,.84,.32,1) forwards">
          <svg viewBox="0 0 400 400" aria-hidden="true" style="position:absolute;width:min(430px,100%);overflow:visible">
            <circle cx="200" cy="200" r="188" fill="none" stroke="#F7F4EF" stroke-opacity=".12" stroke-width="1" stroke-dasharray="4 8" />
            <circle cx="200" cy="200" r="152" fill="none" stroke="#FF6A00" stroke-opacity=".3" stroke-width="1" stroke-dasharray="1180" stroke-dashoffset="1180" style="animation:lhDraw 2.4s .5s cubic-bezier(.3,.8,.3,1) forwards" />
            <g fill="none" stroke="#FF6A00" stroke-width="1.2" stroke-dasharray="240" stroke-dashoffset="240" style="animation:lhDraw 1.7s .9s cubic-bezier(.3,.8,.3,1) forwards">
              <path d="M4 200 H58 V246 H100" />
              <path d="M396 200 H342 V154 H300" />
            </g>
            <circle cx="4" cy="200" r="4" fill="#FF6A00" style="animation:lhNode 2.6s 1.5s ease-in-out infinite" />
            <circle cx="396" cy="200" r="4" fill="#6E90E8" style="animation:lhNode 2.6s 1.8s ease-in-out infinite" />
          </svg>
          <div style="position:relative;display:grid;place-items:center;width:min(300px,74%);aspect-ratio:1;border-radius:50%;background:#F7F4EF;box-shadow:0 0 0 1px rgba(255,138,43,.4),0 0 110px rgba(255,106,0,.55),0 34px 80px rgba(0,0,0,.55);animation:lhFloat 8s ease-in-out infinite">
            <img src={lhhLogo} alt="Longhorn Hacks circuit longhorn logo" width="240" height="240" style="width:76%;height:auto" />
          </div>

          <div data-hcard="1" style="position:absolute;top:4%;left:-6%;display:flex;align-items:center;gap:12px;padding:11px 15px;border-radius:14px;border:1px solid rgba(247,244,239,.14);background:rgba(9,16,38,.82);backdrop-filter:blur(14px);box-shadow:0 16px 40px rgba(0,0,0,.45);animation:lhFloat 6.5s .4s ease-in-out infinite">
            <span style="display:grid;place-items:center;width:30px;height:30px;border-radius:9px;background:rgba(255,106,0,.16);color:#FF8A2B;font-family:'JetBrains Mono',monospace;font-size:12px">✳</span>
            <span style="display:flex;flex-direction:column;gap:3px"><span style="font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.16em;color:#7D87A5">STUDENTS REACHED</span><span style="font-family:'JetBrains Mono',monospace;font-weight:500;font-size:11px;color:#FF8A2B">&lt;STATS GO HERE&gt;</span></span>
          </div>
          <div data-hcard="1" style="position:absolute;bottom:12%;right:-8%;display:flex;align-items:center;gap:12px;padding:11px 15px;border-radius:14px;border:1px solid rgba(247,244,239,.14);background:rgba(9,16,38,.82);backdrop-filter:blur(14px);box-shadow:0 16px 40px rgba(0,0,0,.45);animation:lhFloat 7.5s 1.1s ease-in-out infinite">
            <span style="display:grid;place-items:center;width:30px;height:30px;border-radius:9px;background:rgba(110,144,232,.16);color:#8FAAF2;font-family:'JetBrains Mono',monospace;font-size:12px">◈</span>
            <span style="display:flex;flex-direction:column;gap:3px"><span style="font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.16em;color:#7D87A5">DEVICES DONATED</span><span style="font-family:'JetBrains Mono',monospace;font-weight:500;font-size:11px;color:#8FAAF2">&lt;STATS GO HERE&gt;</span></span>
          </div>
          <div data-hcard="1" style="position:absolute;bottom:-2%;left:2%;transform:rotate(-6deg);padding:8px 14px;border-radius:999px;background:#F7F4EF;color:#04091A;border:2px solid #04091A;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;box-shadow:0 10px 26px rgba(0,0,0,.4)">FREE — ALWAYS</div>
        </div>
      </div>

      <div style="display:flex;align-items:center;justify-content:space-between;gap:20px;margin-top:clamp(34px,4.4vw,60px);opacity:0;animation:lhRise .9s .62s cubic-bezier(.16,.84,.32,1) forwards">
        <a href="#mission" style="display:inline-flex;align-items:center;gap:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.18em;text-transform:uppercase;color:#8B93AE;transition:color .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp4"><span style="display:grid;place-items:center;width:38px;height:38px;border-radius:50%;border:1px solid rgba(247,244,239,.2);font-size:13px">↓</span>Scroll to explore</a>
        <span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.18em;color:#5E688A;text-align:right">EST. AUSTIN / TX</span>
      </div>
    </div>
  </section>

  <div style="position:relative;overflow:hidden;background:#FF6A00;color:#04091A;padding:14px 0;border-top:1px solid rgba(4,9,26,.2);border-bottom:1px solid rgba(4,9,26,.2)">
    <div style="display:flex;gap:0;width:max-content;animation:lhMarquee 34s linear infinite">
      <div style="display:flex;align-items:center;gap:26px;padding-right:26px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:15px;letter-spacing:.04em;text-transform:uppercase;white-space:nowrap">
        <span>Free CS &amp; AI lessons</span><span>✳</span><span>Device donations</span><span>✳</span><span>Hackathons</span><span>✳</span><span>Summer programs</span><span>✳</span><span>Nonprofit partnerships</span><span>✳</span>
      </div>
      <div aria-hidden="true" style="display:flex;align-items:center;gap:26px;padding-right:26px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:15px;letter-spacing:.04em;text-transform:uppercase;white-space:nowrap">
        <span>Free CS &amp; AI lessons</span><span>✳</span><span>Device donations</span><span>✳</span><span>Hackathons</span><span>✳</span><span>Summer programs</span><span>✳</span><span>Nonprofit partnerships</span><span>✳</span>
      </div>
    </div>
  </div>

  <section id="mission" style="position:relative;overflow:hidden;background:#04091A;padding:clamp(72px,9vw,132px) clamp(18px,3.4vw,40px)">
    <div style="position:absolute;top:0;left:0;right:0;height:1px;background:rgba(247,244,239,.12)"></div>
    <div style="position:absolute;top:0;left:clamp(18px,3.4vw,40px);width:clamp(90px,13vw,190px);height:3px;background:#FF6A00"></div>
    <div style="max-width:1340px;margin:0 auto;position:relative">
      <div aria-hidden="true" data-para="0.16" style="position:absolute;top:-40px;right:0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 120;font-weight:800;font-size:clamp(90px,17vw,240px);line-height:.8;color:rgba(247,244,239,.035);pointer-events:none;user-select:none">01</div>
      <div data-reveal="up" style="display:flex;align-items:center;gap:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.22em;color:#FF8A2B;margin-bottom:clamp(28px,3.4vw,44px)"><span style="width:22px;height:1px;background:#FF6A00"></span>//MISSION</div>
      <p data-reveal="mask" style="position:relative;margin:0;max-width:22ch;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(28px,4.6vw,64px);line-height:1.04;letter-spacing:-.02em;text-transform:uppercase">Technology <span style="color:#FF6A00">literacy</span> is not a privilege.</p>
      <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));gap:clamp(24px,4vw,64px);margin-top:clamp(36px,4.6vw,60px);padding-top:28px;border-top:1px solid rgba(247,244,239,.12)">
        <p data-reveal="up" style="margin:0;font-size:16.5px;line-height:1.7;color:#B9C0D4">Longhorn Hacks is a registered 501(c)(3) nonprofit dedicated to expanding access to technology education and creating opportunities for underserved communities.</p>
        <p data-reveal="up" data-reveal-delay="120" style="margin:0;font-size:16.5px;line-height:1.7;color:#B9C0D4">Through corporate sponsorships and fundraising events, we bridge the digital divide by increasing computer science and AI literacy — connecting people through technology and empowering the next generation of leaders.</p>
        <a data-reveal="up" data-reveal-delay="220" href="#/about" style="align-self:start;display:inline-flex;align-items:center;gap:12px;padding:8px 8px 8px 20px;border-radius:999px;border:1px solid rgba(247,244,239,.18);color:#F7F4EF;font-family:'JetBrains Mono',monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;transition:border-color .3s ease,color .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp5">Our story<span style="display:grid;place-items:center;width:30px;height:30px;border-radius:50%;background:rgba(247,244,239,.08);font-size:12px">→</span></a>
      </div>
    </div>
  </section>

  <section style="position:relative;background:#04091A;padding:0 clamp(18px,3.4vw,40px) clamp(72px,9vw,132px)">
    <div style="max-width:1340px;margin:0 auto">
      <div style="display:flex;flex-wrap:wrap;gap:20px;align-items:flex-end;justify-content:space-between;margin-bottom:clamp(28px,3.4vw,44px)">
        <div>
          <div data-reveal="up" style="display:flex;align-items:center;gap:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.22em;color:#FF8A2B;margin-bottom:20px"><span style="width:22px;height:1px;background:#FF6A00"></span>//WHAT WE DO</div>
          <h2 data-reveal="up" data-reveal-delay="80" style="margin:0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(26px,3.6vw,50px);line-height:1;letter-spacing:-.015em;text-transform:uppercase;max-width:16ch">Four pillars, <span style="color:#FF6A00">one outcome</span></h2>
        </div>
        <p data-reveal="up" data-reveal-delay="140" style="margin:0;max-width:34ch;font-size:15.5px;line-height:1.62;color:#8B93AE">Teach it, equip it, celebrate it, then scale it with partners who reach further than we can alone.</p>
      </div>

      <div id="lh-bento" style="display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:14px">
        <article data-reveal="up" style="position:relative;overflow:hidden;grid-column:span 2;padding:30px 28px 32px;border-radius:18px;border:1px solid rgba(247,244,239,.11);background:rgba(255,106,0,.1);min-height:236px;display:flex;flex-direction:column;justify-content:space-between;transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scp6">
          <div style="position:absolute;inset:0;opacity:.4;pointer-events:none;background-image:radial-gradient(rgba(247,244,239,.14) 1px,transparent 1px);background-size:22px 22px"></div>
          <div style="position:relative;display:flex;align-items:flex-start;justify-content:space-between;gap:16px">
            <span style="display:grid;place-items:center;width:40px;height:40px;border-radius:11px;background:rgba(255,106,0,.18);border:1px solid rgba(255,138,43,.34);color:#FF8A2B;font-family:'JetBrains Mono',monospace;font-size:14px">&lt;/&gt;</span>
            <span style="font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#7D87A5">01 / EDUCATE</span>
          </div>
          <div style="position:relative">
            <h3 style="margin:0 0 10px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(20px,2.2vw,28px);letter-spacing:-.01em;text-transform:uppercase">Free CS &amp; AI lessons</h3>
            <p style="margin:0;max-width:40ch;color:#9AA3BC;font-size:15px;line-height:1.6">Hands-on classes that take a student from their first loop to their first model. No cost, no prerequisites, no laptop required.</p>
          </div>
        </article>

        <article data-reveal="up" data-reveal-delay="80" style="position:relative;overflow:hidden;grid-column:span 2;padding:30px 28px 32px;border-radius:18px;border:1px solid rgba(247,244,239,.11);background:rgba(9,16,38,.6);min-height:236px;display:flex;flex-direction:column;justify-content:space-between;transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scp6">
          <div style="display:flex;align-items:flex-start;justify-content:space-between;gap:16px">
            <span style="display:grid;place-items:center;width:40px;height:40px;border-radius:11px;background:rgba(110,144,232,.16);border:1px solid rgba(110,144,232,.3);color:#8FAAF2;font-family:'JetBrains Mono',monospace;font-size:14px">▤</span>
            <span style="font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#7D87A5">02 / EQUIP</span>
          </div>
          <div>
            <h3 style="margin:0 0 10px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(20px,2.2vw,28px);letter-spacing:-.01em;text-transform:uppercase">Technology donations</h3>
            <p style="margin:0;max-width:40ch;color:#9AA3BC;font-size:15px;line-height:1.6">Fundraising that becomes laptops, kits, and connectivity for rural and underfunded classrooms.</p>
          </div>
        </article>

        <article data-reveal="up" data-reveal-delay="160" style="position:relative;overflow:hidden;grid-column:span 1;padding:26px 24px 28px;border-radius:18px;border:1px solid rgba(247,244,239,.11);background:rgba(9,16,38,.6);min-height:210px;display:flex;flex-direction:column;justify-content:space-between;transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scp6">
          <span style="display:grid;place-items:center;width:38px;height:38px;border-radius:11px;background:rgba(255,106,0,.14);border:1px solid rgba(255,138,43,.28);color:#FF8A2B;font-family:'JetBrains Mono',monospace;font-size:14px">⚡</span>
          <div>
            <span style="display:block;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#7D87A5;margin-bottom:10px">03 / IGNITE</span>
            <h3 style="margin:0 0 8px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:19px;letter-spacing:-.01em;text-transform:uppercase">Hackathons</h3>
            <p style="margin:0;color:#9AA3BC;font-size:14.5px;line-height:1.58">Immersive events where students ship something real.</p>
          </div>
        </article>

        <article data-reveal="up" data-reveal-delay="240" style="position:relative;overflow:hidden;grid-column:span 1;padding:26px 24px 28px;border-radius:18px;border:1px solid rgba(247,244,239,.11);background:rgba(9,16,38,.6);min-height:210px;display:flex;flex-direction:column;justify-content:space-between;transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scp6">
          <span style="display:grid;place-items:center;width:38px;height:38px;border-radius:11px;background:rgba(110,144,232,.14);border:1px solid rgba(110,144,232,.28);color:#8FAAF2;font-family:'JetBrains Mono',monospace;font-size:14px">◇</span>
          <div>
            <span style="display:block;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#7D87A5;margin-bottom:10px">04 / SCALE</span>
            <h3 style="margin:0 0 8px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:19px;letter-spacing:-.01em;text-transform:uppercase">Partnerships</h3>
            <p style="margin:0;color:#9AA3BC;font-size:14.5px;line-height:1.58">One curriculum, carried statewide by partner nonprofits.</p>
          </div>
        </article>

        <article data-reveal="up" data-reveal-delay="320" style="position:relative;overflow:hidden;grid-column:span 2;padding:26px 28px;border-radius:18px;border:1px solid rgba(255,138,43,.3);background:rgba(255,106,0,.12);min-height:210px;display:flex;flex-direction:column;justify-content:center;gap:16px">
          <span style="font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#FFA65C">THE OUTCOME</span>
          <p style="margin:0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:clamp(18px,1.9vw,24px);line-height:1.22;letter-spacing:-.01em">A student who walks in never having coded, and walks out with a project, a laptop, and a reason to keep going.</p>
        </article>
      </div>
    </div>
  </section>

  <section style="position:relative;overflow:hidden;background:#060D24;padding:clamp(72px,9vw,124px) clamp(18px,3.4vw,40px);border-top:1px solid rgba(247,244,239,.08);border-bottom:1px solid rgba(247,244,239,.08)">
    <div style="position:absolute;inset:0;opacity:.45;pointer-events:none;background-image:radial-gradient(rgba(247,244,239,.13) 1px,transparent 1px);background-size:28px 28px"></div>
    <div style="max-width:1340px;margin:0 auto;position:relative">
      <div id="lh-orbitgrid" style="display:grid;grid-template-columns:minmax(0,1fr) minmax(0,1fr);gap:clamp(32px,5vw,72px);align-items:center">
        <div>
          <div data-reveal="up" style="display:flex;align-items:center;gap:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.22em;color:#FF8A2B;margin-bottom:20px"><span style="width:22px;height:1px;background:#FF6A00"></span>//WHERE IT GOES</div>
          <h2 data-reveal="up" data-reveal-delay="80" style="margin:0 0 30px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(26px,3.6vw,50px);line-height:1;letter-spacing:-.015em;text-transform:uppercase;max-width:14ch">How a dollar becomes a <span style="color:#FF6A00">developer</span></h2>
          <div data-reveal="up" data-reveal-delay="160" style="display:flex;flex-direction:column;gap:8px">
            <div data-step="1" style="display:flex;align-items:center;gap:16px;padding:16px 18px;border-radius:13px;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .35s ease,background .35s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp7">
              <span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B">01</span>
              <span style="flex:1"><span style="display:block;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:16px;text-transform:uppercase">Sponsors fund a cohort</span><span style="display:block;margin-top:5px;color:#8B93AE;font-size:14px">Corporate sponsorships and fundraising events cover an entire class.</span></span>
            </div>
            <div data-step="1" style="display:flex;align-items:center;gap:16px;padding:16px 18px;border-radius:13px;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .35s ease,background .35s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp7">
              <span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B">02</span>
              <span style="flex:1"><span style="display:block;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:16px;text-transform:uppercase">We buy the hardware</span><span style="display:block;margin-top:5px;color:#8B93AE;font-size:14px">Laptops and kits go directly to students who keep them.</span></span>
            </div>
            <div data-step="1" style="display:flex;align-items:center;gap:16px;padding:16px 18px;border-radius:13px;border:1px solid rgba(255,138,43,.55);background:rgba(255,106,0,.1);transition:border-color .35s ease,background .35s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp8">
              <span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B">03</span>
              <span style="flex:1"><span style="display:block;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:16px;text-transform:uppercase">Students learn free</span><span style="display:block;margin-top:5px;color:#C6CCDD;font-size:14px">Workshops, summer intensives, and hackathons at zero cost.</span></span>
            </div>
            <div data-step="1" style="display:flex;align-items:center;gap:16px;padding:16px 18px;border-radius:13px;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .35s ease,background .35s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp7">
              <span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B">04</span>
              <span style="flex:1"><span style="display:block;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:16px;text-transform:uppercase">Partners multiply it</span><span style="display:block;margin-top:5px;color:#8B93AE;font-size:14px">The same curriculum runs in other communities through partners.</span></span>
            </div>
          </div>
        </div>

        <div data-reveal="up" data-reveal-delay="120" style="position:relative;display:grid;place-items:center;aspect-ratio:1;max-width:480px;justify-self:center;width:100%">
          <div style="position:absolute;width:96%;aspect-ratio:1;border-radius:50%;border:1px dashed rgba(247,244,239,.16);animation:lhSpin 60s linear infinite"></div>
          <div style="position:absolute;width:68%;aspect-ratio:1;border-radius:50%;border:1px solid rgba(255,106,0,.24);animation:lhSpin 44s linear infinite reverse"></div>
          <div style="position:absolute;width:40%;aspect-ratio:1;border-radius:50%;border:1px solid rgba(255,138,43,.55);animation:lhPulse 4.5s ease-in-out infinite"></div>
          <div style="position:relative;display:grid;place-items:center;width:34%;aspect-ratio:1;border-radius:50%;background:#F7F4EF;box-shadow:0 0 60px rgba(255,106,0,.5)">
            <img src={lhhLogo} alt="" width="120" height="120" style="width:76%;height:auto" />
          </div>
          <span style="position:absolute;top:4%;left:50%;transform:translateX(-50%);display:grid;place-items:center;width:56px;height:56px;border-radius:16px;border:1px solid rgba(247,244,239,.16);background:rgba(9,16,38,.9);color:#8FAAF2;font-family:'JetBrains Mono',monospace;font-size:15px">$</span>
          <span style="position:absolute;right:2%;top:50%;transform:translateY(-50%);display:grid;place-items:center;width:56px;height:56px;border-radius:16px;border:1px solid rgba(247,244,239,.16);background:rgba(9,16,38,.9);color:#8FAAF2;font-family:'JetBrains Mono',monospace;font-size:15px">▤</span>
          <span style="position:absolute;bottom:4%;left:50%;transform:translateX(-50%);display:grid;place-items:center;width:64px;height:64px;border-radius:18px;border:1px solid rgba(255,138,43,.6);background:#FF6A00;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:17px;box-shadow:0 0 44px rgba(255,106,0,.7)">&lt;/&gt;</span>
          <span style="position:absolute;left:2%;top:50%;transform:translateY(-50%);display:grid;place-items:center;width:56px;height:56px;border-radius:16px;border:1px solid rgba(247,244,239,.16);background:rgba(9,16,38,.9);color:#8FAAF2;font-family:'JetBrains Mono',monospace;font-size:15px">◇</span>
        </div>
      </div>
    </div>
  </section>

  <section style="position:relative;background:#04091A;padding:clamp(64px,8vw,110px) clamp(18px,3.4vw,40px)">
    <div style="max-width:1340px;margin:0 auto">
      <div data-reveal="up" style="display:flex;flex-wrap:wrap;gap:14px;align-items:baseline;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.22em;color:#FF8A2B;margin-bottom:clamp(32px,4vw,52px)"><span style="width:22px;height:1px;background:#FF6A00;align-self:center"></span>//IMPACT TO DATE<span style="color:#5E688A;letter-spacing:.14em">[PLACEHOLDER FIGURES]</span></div>
      <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:14px">
        <div data-reveal="up" style="position:relative;padding:26px 22px 24px;border-radius:16px;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5)">
          <div style="font-family:'JetBrains Mono',monospace;font-weight:500;font-size:clamp(14px,1.5vw,19px);letter-spacing:.04em;line-height:1.2;color:#F7F4EF">&lt;STATS GO HERE&gt;</div>
          <div style="margin-top:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.16em;color:#7D87A5">STUDENTS REACHED</div>
        </div>
        <div data-reveal="up" data-reveal-delay="80" style="position:relative;padding:26px 22px 24px;border-radius:16px;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5)">
          <div style="font-family:'JetBrains Mono',monospace;font-weight:500;font-size:clamp(14px,1.5vw,19px);letter-spacing:.04em;line-height:1.2;color:#F7F4EF">&lt;STATS GO HERE&gt;</div>
          <div style="margin-top:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.16em;color:#7D87A5">WORKSHOPS HOSTED</div>
        </div>
        <div data-reveal="up" data-reveal-delay="160" style="position:relative;padding:26px 22px 24px;border-radius:16px;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5)">
          <div style="font-family:'JetBrains Mono',monospace;font-weight:500;font-size:clamp(14px,1.5vw,19px);letter-spacing:.04em;line-height:1.2;color:#F7F4EF">&lt;STATS GO HERE&gt;</div>
          <div style="margin-top:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.16em;color:#7D87A5">COMMUNITIES SERVED</div>
        </div>
        <div data-reveal="up" data-reveal-delay="240" style="position:relative;padding:26px 22px 24px;border-radius:16px;border:1px solid rgba(255,138,43,.34);background:rgba(255,106,0,.12)">
          <div style="font-family:'JetBrains Mono',monospace;font-weight:500;font-size:clamp(14px,1.5vw,19px);letter-spacing:.04em;line-height:1.2;color:#FF8A2B">&lt;STATS GO HERE&gt;</div>
          <div style="margin-top:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.16em;color:#FFA65C">DEVICES DONATED</div>
        </div>
      </div>
    </div>
  </section>

  <section style="position:relative;background:#04091A;padding:clamp(40px,5vw,64px) clamp(18px,3.4vw,40px) clamp(72px,9vw,124px)">
    <div style="max-width:1340px;margin:0 auto">
      <div style="display:flex;flex-wrap:wrap;gap:20px;align-items:flex-end;justify-content:space-between;margin-bottom:clamp(28px,3.4vw,44px)">
        <div>
          <div data-reveal="up" style="display:flex;align-items:center;gap:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.22em;color:#FF8A2B;margin-bottom:20px"><span style="width:22px;height:1px;background:#FF6A00"></span>//PROGRAMS</div>
          <h2 data-reveal="up" data-reveal-delay="80" style="margin:0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(26px,3.6vw,50px);line-height:1;letter-spacing:-.015em;text-transform:uppercase;max-width:16ch">Built for students who were never <span style="color:#FF6A00">on the list</span></h2>
        </div>
        <a data-reveal="up" data-reveal-delay="140" href="#/programs" style="display:inline-flex;align-items:center;gap:12px;padding:8px 8px 8px 20px;border-radius:999px;border:1px solid rgba(247,244,239,.18);color:#F7F4EF;font-family:'JetBrains Mono',monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;transition:border-color .3s ease,color .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp5">All programs<span style="display:grid;place-items:center;width:30px;height:30px;border-radius:50%;background:rgba(247,244,239,.08);font-size:12px">→</span></a>
      </div>
      <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(260px,1fr));gap:14px">
        <article data-reveal="up" style="position:relative;overflow:hidden;border-radius:18px;border:1px solid rgba(247,244,239,.11);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scp9">
          <div style="position:relative;aspect-ratio:4/3;background:#0A1330"><image-slot id="lhh-p1" shape="rect" placeholder="Workshop photo"></image-slot></div>
          <div style="padding:24px 24px 26px">
            <div style="display:flex;align-items:center;gap:10px;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#FF8A2B">01<span style="color:#7D87A5">WORKSHOPS</span></div>
            <h3 style="margin:14px 0 10px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:20px;letter-spacing:-.01em;text-transform:uppercase">Community workshops</h3>
            <p style="margin:0;color:#9AA3BC;font-size:14.5px;line-height:1.6">Weekend sessions at libraries and schools. Students leave with a project and a laptop.</p>
          </div>
        </article>
        <article data-reveal="up" data-reveal-delay="110" style="position:relative;overflow:hidden;border-radius:18px;border:1px solid rgba(247,244,239,.11);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scp9">
          <div style="position:relative;aspect-ratio:4/3;background:#0A1330"><image-slot id="lhh-p2" shape="rect" placeholder="Summer program photo"></image-slot></div>
          <div style="padding:24px 24px 26px">
            <div style="display:flex;align-items:center;gap:10px;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#FF8A2B">02<span style="color:#7D87A5">SUMMER</span></div>
            <h3 style="margin:14px 0 10px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:20px;letter-spacing:-.01em;text-transform:uppercase">Summer intensives</h3>
            <p style="margin:0;color:#9AA3BC;font-size:14.5px;line-height:1.6">Multi-week programs in data structures and machine learning, ending in a capstone.</p>
          </div>
        </article>
        <article data-reveal="up" data-reveal-delay="220" style="position:relative;overflow:hidden;border-radius:18px;border:1px solid rgba(247,244,239,.11);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scp9">
          <div style="position:relative;aspect-ratio:4/3;background:#0A1330"><image-slot id="lhh-p3" shape="rect" placeholder="Hackathon photo"></image-slot></div>
          <div style="padding:24px 24px 26px">
            <div style="display:flex;align-items:center;gap:10px;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#FF8A2B">03<span style="color:#7D87A5">EVENTS</span></div>
            <h3 style="margin:14px 0 10px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:20px;letter-spacing:-.01em;text-transform:uppercase">Hackathons &amp; partnerships</h3>
            <p style="margin:0;color:#9AA3BC;font-size:14.5px;line-height:1.6">Our own events plus partner networks, reaching students across the state.</p>
          </div>
        </article>
      </div>
    </div>
  </section>

  <section style="position:relative;overflow:hidden;background:#060D24;padding:clamp(72px,9vw,124px) clamp(18px,3.4vw,40px);border-top:1px solid rgba(247,244,239,.08)">
    <div style="max-width:1340px;margin:0 auto">
      <div style="display:flex;flex-wrap:wrap;gap:20px;align-items:flex-end;justify-content:space-between;margin-bottom:clamp(30px,3.6vw,48px)">
        <div>
          <div data-reveal="up" style="display:flex;align-items:center;gap:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.22em;color:#FF8A2B;margin-bottom:20px"><span style="width:22px;height:1px;background:#FF6A00"></span>//LEADERSHIP</div>
          <h2 data-reveal="up" data-reveal-delay="80" style="margin:0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(26px,3.6vw,50px);line-height:1;letter-spacing:-.015em;text-transform:uppercase">The people behind it</h2>
        </div>
        <p data-reveal="up" data-reveal-delay="140" style="margin:0;max-width:32ch;color:#8B93AE;font-size:15px;line-height:1.6">Students and organizers running curriculum, partnerships, and everything in between.</p>
      </div>
      <div style="display:grid;grid-template-columns:repeat(auto-fill,minmax(178px,1fr));gap:12px">
        <article data-reveal="up" style="border-radius:14px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scpa"><div style="aspect-ratio:1;background:#0A1330"><image-slot id="lhh-t1" shape="rect" placeholder="Varsha A."></image-slot></div><div style="padding:15px 16px 17px"><div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:15.5px;text-transform:uppercase">Varsha A.</div><div style="margin-top:7px;font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.14em;color:#FF8A2B">CO-PRESIDENT</div></div></article>
        <article data-reveal="up" data-reveal-delay="60" style="border-radius:14px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scpa"><div style="aspect-ratio:1;background:#0A1330"><image-slot id="lhh-t2" shape="rect" placeholder="Leslie C."></image-slot></div><div style="padding:15px 16px 17px"><div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:15.5px;text-transform:uppercase">Leslie C.</div><div style="margin-top:7px;font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.14em;color:#FF8A2B">CO-PRESIDENT</div></div></article>
        <article data-reveal="up" data-reveal-delay="120" style="border-radius:14px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scpa"><div style="aspect-ratio:1;background:#0A1330"><image-slot id="lhh-t3" shape="rect" placeholder="Vihaan K."></image-slot></div><div style="padding:15px 16px 17px"><div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:15.5px;text-transform:uppercase">Vihaan K.</div><div style="margin-top:7px;font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.14em;color:#FF8A2B">SECRETARY</div></div></article>
        <article data-reveal="up" data-reveal-delay="180" style="border-radius:14px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scpa"><div style="aspect-ratio:1;background:#0A1330"><image-slot id="lhh-t4" shape="rect" placeholder="Anishka K."></image-slot></div><div style="padding:15px 16px 17px"><div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:15.5px;text-transform:uppercase">Anishka K.</div><div style="margin-top:7px;font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.14em;color:#FF8A2B">TREASURER</div></div></article>
        <article data-reveal="up" data-reveal-delay="240" style="border-radius:14px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scpa"><div style="aspect-ratio:1;background:#0A1330"><image-slot id="lhh-t5" shape="rect" placeholder="Aarjit [last name]"></image-slot></div><div style="padding:15px 16px 17px"><div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:15.5px;text-transform:uppercase">Aarjit [last name]</div><div style="margin-top:7px;font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.14em;color:#FF8A2B">DIR. OF CURRICULUM</div></div></article>
        <article data-reveal="up" data-reveal-delay="300" style="border-radius:14px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scpa"><div style="aspect-ratio:1;background:#0A1330"><image-slot id="lhh-t6" shape="rect" placeholder="Connor E."></image-slot></div><div style="padding:15px 16px 17px"><div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:15.5px;text-transform:uppercase">Connor E.</div><div style="margin-top:7px;font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.14em;color:#FF8A2B">DIR. OF CURRICULUM</div></div></article>
        <article data-reveal="up" data-reveal-delay="360" style="border-radius:14px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scpa"><div style="aspect-ratio:1;background:#0A1330"><image-slot id="lhh-t7" shape="rect" placeholder="Jovan N."></image-slot></div><div style="padding:15px 16px 17px"><div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:15.5px;text-transform:uppercase">Jovan N.</div><div style="margin-top:7px;font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.14em;color:#FF8A2B">DIR. OF PARTNERSHIPS</div></div></article>
        <article data-reveal="up" data-reveal-delay="420" style="border-radius:14px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scpa"><div style="aspect-ratio:1;background:#0A1330"><image-slot id="lhh-t8" shape="rect" placeholder="Namish J."></image-slot></div><div style="padding:15px 16px 17px"><div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:15.5px;text-transform:uppercase">Namish J.</div><div style="margin-top:7px;font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.14em;color:#FF8A2B">DIR. OF PARTNERSHIPS</div></div></article>
        <article data-reveal="up" data-reveal-delay="480" style="border-radius:14px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:rgba(9,16,38,.5);transition:border-color .4s ease,transform .4s cubic-bezier(.2,.7,.2,1)" class="scpa"><div style="aspect-ratio:1;background:#0A1330"><image-slot id="lhh-t9" shape="rect" placeholder="Luana T."></image-slot></div><div style="padding:15px 16px 17px"><div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:15.5px;text-transform:uppercase">Luana T.</div><div style="margin-top:7px;font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.14em;color:#FF8A2B">DIR. OF MARKETING</div></div></article>
      </div>
    </div>
  </section>

  <section style="position:relative;overflow:hidden;background:#04091A;padding:clamp(72px,9vw,124px) clamp(18px,3.4vw,40px)">
    <div style="max-width:1340px;margin:0 auto;position:relative">
      <div style="position:relative;overflow:hidden;border-radius:26px;border:1px solid rgba(255,138,43,.32);background:#FF6A00;color:#04091A;padding:clamp(38px,5.4vw,76px) clamp(24px,4vw,64px)">
        <div aria-hidden="true" data-para="0.1" style="position:absolute;right:-2%;bottom:-14%;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 122;font-weight:800;font-size:clamp(80px,15vw,210px);line-height:.8;color:rgba(4,9,26,.16);pointer-events:none;user-select:none;white-space:nowrap">LHH</div>
        <div style="position:relative;display:grid;grid-template-columns:repeat(auto-fit,minmax(290px,1fr));gap:clamp(28px,4vw,64px);align-items:end">
          <div>
            <div data-reveal="up" style="display:inline-flex;align-items:center;gap:9px;padding:7px 14px;border-radius:999px;background:#F7F4EF;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.16em;transform:rotate(-2deg);margin-bottom:22px">TAX-DEDUCTIBLE ✳ 501(C)(3)</div>
            <h2 data-reveal="up" data-reveal-delay="80" style="margin:0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 116;font-weight:800;font-size:clamp(32px,5.2vw,74px);line-height:.94;letter-spacing:-.025em;text-transform:uppercase;max-width:13ch">Fund a student's first line of code</h2>
          </div>
          <div data-reveal="up" data-reveal-delay="160">
            <p style="margin:0 0 28px;color:rgba(4,9,26,.82);font-size:16px;line-height:1.65;max-width:42ch">Every dollar becomes laptops, curriculum, and free events for communities left out of the tech economy.</p>
            <div style="display:flex;flex-wrap:wrap;gap:12px">
              <a href="#/contact" style="display:inline-flex;align-items:center;gap:12px;padding:8px 8px 8px 22px;border-radius:999px;background:#F7F4EF;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.14em;text-transform:uppercase;transition:transform .3s cubic-bezier(.2,.7,.2,1),box-shadow .3s ease" class="scpb">Donate now<span style="display:grid;place-items:center;width:32px;height:32px;border-radius:50%;background:#FF6A00;font-size:13px">→</span></a>
              <a href="#/contact" style="display:inline-flex;align-items:center;gap:12px;padding:16px 24px;border-radius:999px;border:1px solid rgba(4,9,26,.45);color:#04091A;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.14em;text-transform:uppercase;transition:background .3s ease,transform .3s cubic-bezier(.2,.7,.2,1)" class="scpc">Partner with us</a>
            </div>
          </div>
        </div>
      </div>

      <div data-reveal="up" data-reveal-delay="220" style="margin-top:clamp(40px,5vw,68px);padding-top:26px;border-top:1px solid rgba(247,244,239,.12)">
        <div style="font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.22em;color:#7D87A5;margin-bottom:22px">OUR SPONSORS — [LOGOS TO BE ADDED]</div>
        <div style="overflow:hidden;mask-image:linear-gradient(90deg,transparent,#000 6%,#000 94%,transparent);-webkit-mask-image:linear-gradient(90deg,transparent,#000 6%,#000 94%,transparent)">
          <div style="display:flex;gap:12px;width:max-content;animation:lhMarquee 40s linear infinite">
            <div style="display:flex;gap:12px">
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_01</div>
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_02</div>
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_03</div>
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_04</div>
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_05</div>
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_06</div>
            </div>
            <div style="display:flex;gap:12px" aria-hidden="true">
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_01</div>
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_02</div>
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_03</div>
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_04</div>
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_05</div>
              <div style="width:172px;height:66px;border:1px solid rgba(247,244,239,.14);border-radius:12px;display:grid;place-items:center;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#5E688A">SPONSOR_06</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</div>

<div data-screen="/about" data-screen-label="About" style="display:none">
  <section style="position:relative;overflow:hidden;background:#04091A;padding:clamp(128px,17vh,190px) clamp(18px,3.4vw,40px) clamp(48px,6vw,72px)">
    <div aria-hidden="true" style="position:absolute;top:-42%;left:50%;transform:translateX(-50%);width:min(1120px,124vw);aspect-ratio:1;border-radius:50%;border:1px solid rgba(255,138,43,.12);border-top:2px solid rgba(255,138,43,.55);pointer-events:none;animation:lhSpin 46s linear infinite"></div>
    <div aria-hidden="true" style="position:absolute;top:-24%;left:50%;transform:translateX(-50%);width:min(760px,92vw);aspect-ratio:1;border-radius:50%;border:1px dashed rgba(255,138,43,.2);pointer-events:none;animation:lhSpin 80s linear infinite reverse"></div>
    <div aria-hidden="true" style="position:absolute;top:clamp(96px,12vh,140px);left:50%;transform:translateX(-50%);width:clamp(80px,10vw,150px);height:4px;background:#FF6A00;pointer-events:none"></div>
    <div style="position:absolute;inset:0;pointer-events:none;opacity:.4;background-image:radial-gradient(rgba(247,244,239,.14) 1px,transparent 1px);background-size:26px 26px;mask-image:linear-gradient(#000,transparent 80%);-webkit-mask-image:linear-gradient(#000,transparent 80%)"></div>
    <div style="position:relative;max-width:1340px;margin:0 auto;text-align:center">
      <div style="display:inline-flex;align-items:center;gap:10px;padding:7px 14px;border-radius:999px;border:1px solid rgba(255,138,43,.32);background:rgba(255,106,0,.09);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.18em;color:#FFA65C" data-reveal="up">02 // ABOUT US</div>
      <h1 data-reveal="up" data-reveal-delay="90" style="margin:clamp(22px,2.8vw,36px) auto 0;max-width:16ch;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 118;font-weight:800;font-size:clamp(36px,6.4vw,92px);line-height:.92;letter-spacing:-.025em;text-transform:uppercase">A nonprofit built by <span style="color:transparent;-webkit-text-stroke:1.4px #FF6A00">students</span>, for students</h1>
      <p data-reveal="up" data-reveal-delay="180" style="margin:clamp(20px,2.4vw,30px) auto 0;max-width:56ch;font-size:16.5px;line-height:1.7;color:#B9C0D4">In the same city that hires thousands of engineers, there are classrooms where no student has ever written a line of code. Access — not talent — is the gap.</p>
      <div data-reveal="up" data-reveal-delay="270" style="display:flex;flex-wrap:wrap;justify-content:center;gap:10px;margin-top:clamp(26px,3vw,38px)">
        <span style="padding:8px 15px;border-radius:999px;background:#F7F4EF;color:#04091A;border:2px solid #04091A;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;transform:rotate(-2deg)">ACCESS FIRST</span>
        <span style="padding:8px 15px;border-radius:999px;background:#FF6A00;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;transform:rotate(1.5deg)">REAL SKILLS</span>
        <span style="padding:8px 15px;border-radius:999px;border:1px solid rgba(247,244,239,.3);color:#F7F4EF;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;transform:rotate(-1deg)">ACCOUNTABLE</span>
      </div>
    </div>
  </section>

  <section style="position:relative;background:#04091A;padding:0 clamp(18px,3.4vw,40px) clamp(64px,8vw,110px)">
    <div style="max-width:1340px;margin:0 auto;display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:12px">
      <div data-reveal="up" style="aspect-ratio:4/5;border-radius:16px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:#0A1330"><image-slot id="lhh-a1" shape="rect" placeholder="Team / classroom photo"></image-slot></div>
      <div data-reveal="up" data-reveal-delay="90" style="aspect-ratio:4/5;border-radius:16px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:#0A1330;margin-top:clamp(0px,3vw,44px)"><image-slot id="lhh-a2" shape="rect" placeholder="Workshop photo"></image-slot></div>
      <div data-reveal="up" data-reveal-delay="180" style="aspect-ratio:4/5;border-radius:16px;overflow:hidden;border:1px solid rgba(247,244,239,.1);background:#0A1330"><image-slot id="lhh-a3" shape="rect" placeholder="Students photo"></image-slot></div>
      <div data-reveal="up" data-reveal-delay="270" style="aspect-ratio:4/5;border-radius:16px;overflow:hidden;border:1px solid rgba(255,138,43,.3);background:rgba(255,106,0,.14);margin-top:clamp(0px,3vw,44px);padding:26px;display:flex;flex-direction:column;justify-content:space-between">
        <span style="font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#FFA65C">EST. AUSTIN</span>
        <p style="margin:0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:19px;line-height:1.25;text-transform:uppercase">We teach for free, and we leave the hardware behind.</p>
      </div>
    </div>
  </section>

  <section style="position:relative;overflow:hidden;background:#060D24;padding:clamp(64px,8vw,110px) clamp(18px,3.4vw,40px);border-top:1px solid rgba(247,244,239,.08)">
    <div aria-hidden="true" style="position:absolute;inset:-120px;pointer-events:none;opacity:.4;background-image:radial-gradient(rgba(247,244,239,.16) 1px,transparent 1px);background-size:24px 24px;animation:lhDrift 26s linear infinite"></div>
    <div aria-hidden="true" data-grow="x-right" style="position:absolute;z-index:0;top:0;left:0;right:0;height:clamp(90px,14vh,180px);background:rgba(255,106,0,.09);border-bottom:1px solid rgba(255,138,43,.4);transform:scaleX(0);transform-origin:right center;pointer-events:none"></div>
    <div style="position:relative;z-index:1;max-width:1340px;margin:0 auto">
      <div data-reveal="up" style="display:flex;align-items:center;gap:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.22em;color:#FF8A2B;margin-bottom:clamp(28px,3.4vw,44px)"><span style="width:22px;height:1px;background:#FF6A00"></span>//WHAT WE STAND FOR</div>
      <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:14px">
        <div data-reveal="up" style="padding:30px 26px 32px;border-radius:18px;border:1px solid rgba(247,244,239,.11);background:rgba(9,16,38,.5)">
          <div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 118;font-weight:800;font-size:44px;line-height:1;color:rgba(255,138,43,.32);margin-bottom:20px">01</div>
          <h3 style="margin:0 0 10px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:19px;text-transform:uppercase">Access first</h3>
          <p style="margin:0;color:#9AA3BC;font-size:15px;line-height:1.62">Free, always. No application fees, no prerequisites, no laptop of your own required.</p>
        </div>
        <div data-reveal="up" data-reveal-delay="100" style="padding:30px 26px 32px;border-radius:18px;border:1px solid rgba(247,244,239,.11);background:rgba(9,16,38,.5)">
          <div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 118;font-weight:800;font-size:44px;line-height:1;color:rgba(255,138,43,.32);margin-bottom:20px">02</div>
          <h3 style="margin:0 0 10px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:19px;text-transform:uppercase">Real skills</h3>
          <p style="margin:0;color:#9AA3BC;font-size:15px;line-height:1.62">Curriculum built by people who write software, not a worksheet packet about it.</p>
        </div>
        <div data-reveal="up" data-reveal-delay="200" style="padding:30px 26px 32px;border-radius:18px;border:1px solid rgba(255,138,43,.34);background:rgba(255,106,0,.12)">
          <div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 118;font-weight:800;font-size:44px;line-height:1;color:#FF8A2B;margin-bottom:20px">03</div>
          <h3 style="margin:0 0 10px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 106;font-weight:700;font-size:19px;text-transform:uppercase">Accountable</h3>
          <p style="margin:0;color:#C6CCDD;font-size:15px;line-height:1.62">Sponsors see exactly where funding lands: devices delivered, students taught, events run.</p>
        </div>
      </div>
    </div>
  </section>

  <section style="position:relative;background:#04091A;padding:clamp(64px,8vw,110px) clamp(18px,3.4vw,40px)">
    <div style="max-width:1340px;margin:0 auto">
      <div data-reveal="up" style="display:flex;align-items:center;gap:12px;font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.22em;color:#FF8A2B;margin-bottom:clamp(26px,3vw,40px)"><span style="width:22px;height:1px;background:#FF6A00"></span>//LEADERSHIP</div>
      <div data-reveal="up" style="display:grid;grid-template-columns:44px minmax(0,1fr) minmax(0,1fr);gap:18px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);transition:padding-left .35s ease,border-color .35s ease" class="lh-trow scpd"><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B;align-self:center">01</span><span style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(17px,1.8vw,23px);text-transform:uppercase">Varsha A.</span><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;color:#7D87A5;align-self:center">CO-PRESIDENT</span></div>
      <div data-reveal="up" data-reveal-delay="50" style="display:grid;grid-template-columns:44px minmax(0,1fr) minmax(0,1fr);gap:18px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);transition:padding-left .35s ease,border-color .35s ease" class="lh-trow scpd"><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B;align-self:center">02</span><span style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(17px,1.8vw,23px);text-transform:uppercase">Leslie C.</span><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;color:#7D87A5;align-self:center">CO-PRESIDENT</span></div>
      <div data-reveal="up" data-reveal-delay="100" style="display:grid;grid-template-columns:44px minmax(0,1fr) minmax(0,1fr);gap:18px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);transition:padding-left .35s ease,border-color .35s ease" class="lh-trow scpd"><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B;align-self:center">03</span><span style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(17px,1.8vw,23px);text-transform:uppercase">Vihaan K.</span><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;color:#7D87A5;align-self:center">SECRETARY</span></div>
      <div data-reveal="up" data-reveal-delay="150" style="display:grid;grid-template-columns:44px minmax(0,1fr) minmax(0,1fr);gap:18px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);transition:padding-left .35s ease,border-color .35s ease" class="lh-trow scpd"><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B;align-self:center">04</span><span style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(17px,1.8vw,23px);text-transform:uppercase">Anishka K.</span><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;color:#7D87A5;align-self:center">TREASURER</span></div>
      <div data-reveal="up" data-reveal-delay="200" style="display:grid;grid-template-columns:44px minmax(0,1fr) minmax(0,1fr);gap:18px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);transition:padding-left .35s ease,border-color .35s ease" class="lh-trow scpd"><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B;align-self:center">05</span><span style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(17px,1.8vw,23px);text-transform:uppercase">Aarjit [last name]</span><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;color:#7D87A5;align-self:center">DIRECTOR OF CURRICULUM</span></div>
      <div data-reveal="up" data-reveal-delay="250" style="display:grid;grid-template-columns:44px minmax(0,1fr) minmax(0,1fr);gap:18px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);transition:padding-left .35s ease,border-color .35s ease" class="lh-trow scpd"><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B;align-self:center">06</span><span style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(17px,1.8vw,23px);text-transform:uppercase">Connor E.</span><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;color:#7D87A5;align-self:center">DIRECTOR OF CURRICULUM</span></div>
      <div data-reveal="up" data-reveal-delay="300" style="display:grid;grid-template-columns:44px minmax(0,1fr) minmax(0,1fr);gap:18px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);transition:padding-left .35s ease,border-color .35s ease" class="lh-trow scpd"><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B;align-self:center">07</span><span style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(17px,1.8vw,23px);text-transform:uppercase">Jovan N.</span><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;color:#7D87A5;align-self:center">DIRECTOR OF PARTNERSHIPS</span></div>
      <div data-reveal="up" data-reveal-delay="350" style="display:grid;grid-template-columns:44px minmax(0,1fr) minmax(0,1fr);gap:18px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);transition:padding-left .35s ease,border-color .35s ease" class="lh-trow scpd"><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B;align-self:center">08</span><span style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(17px,1.8vw,23px);text-transform:uppercase">Namish J.</span><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;color:#7D87A5;align-self:center">DIRECTOR OF PARTNERSHIPS</span></div>
      <div data-reveal="up" data-reveal-delay="400" style="display:grid;grid-template-columns:44px minmax(0,1fr) minmax(0,1fr);gap:18px;padding:15px 0;border-top:1px solid rgba(247,244,239,.12);border-bottom:1px solid rgba(247,244,239,.12);transition:padding-left .35s ease,border-color .35s ease" class="lh-trow scpd"><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#FF8A2B;align-self:center">09</span><span style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 108;font-weight:700;font-size:clamp(17px,1.8vw,23px);text-transform:uppercase">Luana T.</span><span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.14em;color:#7D87A5;align-self:center">DIRECTOR OF MARKETING</span></div>
    </div>
  </section>
</div>

<div data-screen="/programs" data-screen-label="Programs" style="display:none">
  <section style="position:relative;overflow:hidden;background:#04091A;padding:clamp(126px,16vh,180px) clamp(18px,3.4vw,40px) clamp(56px,7vw,88px)">
    <div aria-hidden="true" style="position:absolute;inset:0;pointer-events:none;display:flex;justify-content:flex-end;gap:clamp(26px,5vw,74px);padding-right:clamp(18px,3.4vw,40px)">
      <span style="position:relative;width:1px;background:rgba(247,244,239,.08);overflow:hidden"><span style="position:absolute;inset:0 auto auto 0;width:1px;height:20%;background:#FF8A2B;animation:lhBeam 8s linear infinite"></span></span>
      <span style="position:relative;width:1px;background:rgba(247,244,239,.08);overflow:hidden"><span style="position:absolute;inset:0 auto auto 0;width:1px;height:14%;background:#8FAAF2;animation:lhBeam 11s 2.4s linear infinite"></span></span>
      <span style="position:relative;width:1px;background:rgba(255,138,43,.24);overflow:hidden"><span style="position:absolute;inset:0 auto auto 0;width:1px;height:26%;background:#FF6A00;animation:lhBeam 6.5s 1.2s linear infinite"></span></span>
    </div>
    <div aria-hidden="true" style="position:absolute;inset:0;pointer-events:none;opacity:.35;background-image:radial-gradient(rgba(247,244,239,.14) 1px,transparent 1px);background-size:26px 26px;mask-image:linear-gradient(#000,transparent 78%);-webkit-mask-image:linear-gradient(#000,transparent 78%)"></div>
    <div id="lh-proghero" style="position:relative;max-width:1340px;margin:0 auto;display:grid;grid-template-columns:minmax(0,1.1fr) minmax(0,.9fr);gap:clamp(28px,4vw,64px);align-items:end">
      <div>
        <div style="display:inline-flex;align-items:center;gap:10px;padding:7px 14px;border-radius:999px;border:1px solid rgba(255,138,43,.32);background:rgba(255,106,0,.09);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.18em;color:#FFA65C" data-reveal="up">03 // PROGRAMS</div>
        <h1 data-reveal="up" data-reveal-delay="90" style="margin:clamp(20px,2.6vw,34px) 0 0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 118;font-weight:800;font-size:clamp(36px,6.2vw,88px);line-height:.92;letter-spacing:-.025em;text-transform:uppercase;max-width:12ch">Meet students <span style="color:#FF6A00">where they are</span></h1>
      </div>
      <div data-reveal="right" data-reveal-delay="160" style="display:flex;flex-direction:column;gap:12px">
        <p style="margin:0;font-size:16.5px;line-height:1.68;color:#B9C0D4">Every program is free to attend, taught by working students and engineers, and designed so a participant can keep building the day after it ends.</p>
        <div style="display:flex;flex-wrap:wrap;gap:8px;margin-top:8px">
          <span style="padding:7px 13px;border-radius:999px;border:1px solid rgba(247,244,239,.2);font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#A8B0C8">NO COST</span>
          <span style="padding:7px 13px;border-radius:999px;border:1px solid rgba(247,244,239,.2);font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#A8B0C8">NO PREREQS</span>
          <span style="padding:7px 13px;border-radius:999px;background:#FF6A00;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em">LAPTOPS PROVIDED</span>
        </div>
      </div>
    </div>
  </section>

  <section style="position:relative;overflow:hidden;background:#04091A;padding:0 clamp(18px,3.4vw,40px) clamp(72px,9vw,124px)">
    <div aria-hidden="true" style="position:absolute;inset:0;pointer-events:none;opacity:.5;background-image:repeating-linear-gradient(72deg,rgba(247,244,239,.05) 0 1px,transparent 1px 46px);animation:lhSlide 30s linear infinite"></div>
    <div aria-hidden="true" data-grow="x" style="position:absolute;top:0;left:0;right:0;height:5px;background:#FF6A00;transform:scaleX(0);transform-origin:left center;pointer-events:none"></div>
    <div aria-hidden="true" style="position:absolute;top:0;left:0;right:0;height:1px;background:rgba(247,244,239,.1);pointer-events:none"></div>
    <div style="position:relative;max-width:1340px;margin:0 auto;display:flex;flex-direction:column;gap:14px;padding-top:clamp(26px,3.4vw,48px)">
      <article data-reveal="left" class="lh-prow scpe" style="display:grid;grid-template-columns:minmax(0,.9fr) minmax(0,1.1fr);gap:clamp(20px,3vw,44px);padding:clamp(20px,2.4vw,30px);border-radius:22px;border:1px solid rgba(247,244,239,.11);background:rgba(9,16,38,.5);align-items:center;transition:border-color .4s ease,transform .45s cubic-bezier(.2,.7,.2,1)">
        <div style="aspect-ratio:5/4;border-radius:14px;overflow:hidden;background:#0A1330"><image-slot id="lhh-pp1" shape="rect" placeholder="Workshop photo"></image-slot></div>
        <div>
          <div style="font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#FF8A2B">01 / WORKSHOPS</div>
          <h2 style="margin:16px 0 14px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(24px,3.2vw,42px);line-height:1;letter-spacing:-.015em;text-transform:uppercase">Community workshops</h2>
          <p style="margin:0 0 24px;color:#9AA3BC;font-size:16px;line-height:1.66;max-width:46ch">Weekend sessions hosted at libraries, schools, and community centers. Students leave with a project they built themselves and a laptop to keep building on.</p>
          <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(140px,1fr));gap:10px">
            <span style="padding:12px 14px;border-radius:11px;background:rgba(247,244,239,.05);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.1em;color:#A8B0C8">ONE DAY · 4 HRS</span>
            <span style="padding:12px 14px;border-radius:11px;background:rgba(247,244,239,.05);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.1em;color:#A8B0C8">PYTHON / WEB / AI</span>
            <span style="padding:12px 14px;border-radius:11px;background:rgba(255,106,0,.14);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.1em;color:#FFA65C">FREE ENTRY</span>
          </div>
        </div>
      </article>

      <article data-reveal="right" data-reveal-delay="90" class="lh-prow scpe" style="display:grid;grid-template-columns:minmax(0,1.1fr) minmax(0,.9fr);gap:clamp(20px,3vw,44px);padding:clamp(20px,2.4vw,30px);border-radius:22px;border:1px solid rgba(247,244,239,.11);background:rgba(9,16,38,.5);align-items:center;transition:border-color .4s ease,transform .45s cubic-bezier(.2,.7,.2,1)">
        <div>
          <div style="font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#FF8A2B">02 / SUMMER</div>
          <h2 style="margin:16px 0 14px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(24px,3.2vw,42px);line-height:1;letter-spacing:-.015em;text-transform:uppercase">Summer intensives</h2>
          <p style="margin:0 0 24px;color:#9AA3BC;font-size:16px;line-height:1.66;max-width:46ch">Multi-week programs that go deep — data structures, machine learning fundamentals, and a capstone students present to sponsors and families.</p>
          <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(140px,1fr));gap:10px">
            <span style="padding:12px 14px;border-radius:11px;background:rgba(247,244,239,.05);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.1em;color:#A8B0C8">4 WEEKS</span>
            <span style="padding:12px 14px;border-radius:11px;background:rgba(247,244,239,.05);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.1em;color:#A8B0C8">MENTORED</span>
            <span style="padding:12px 14px;border-radius:11px;background:rgba(247,244,239,.05);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.1em;color:#A8B0C8">CAPSTONE</span>
          </div>
        </div>
        <div style="aspect-ratio:5/4;border-radius:14px;overflow:hidden;background:#0A1330"><image-slot id="lhh-pp2" shape="rect" placeholder="Summer program photo"></image-slot></div>
      </article>

      <article data-reveal="left" data-reveal-delay="180" class="lh-prow scpf" style="display:grid;grid-template-columns:minmax(0,.9fr) minmax(0,1.1fr);gap:clamp(20px,3vw,44px);padding:clamp(20px,2.4vw,30px);border-radius:22px;border:1px solid rgba(255,138,43,.3);background:rgba(255,106,0,.11);align-items:center;transition:border-color .4s ease,transform .45s cubic-bezier(.2,.7,.2,1)">
        <div style="aspect-ratio:5/4;border-radius:14px;overflow:hidden;background:#0A1330"><image-slot id="lhh-pp3" shape="rect" placeholder="Hackathon photo"></image-slot></div>
        <div>
          <div style="font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#FF8A2B">03 / EVENTS</div>
          <h2 style="margin:16px 0 14px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(24px,3.2vw,42px);line-height:1;letter-spacing:-.015em;text-transform:uppercase">Hackathons &amp; partnerships</h2>
          <p style="margin:0 0 24px;color:#C6CCDD;font-size:16px;line-height:1.66;max-width:46ch">We run our own events and plug into partner nonprofits' networks so a single curriculum can reach thousands of students across Texas.</p>
          <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(140px,1fr));gap:10px">
            <span style="padding:12px 14px;border-radius:11px;background:rgba(4,9,26,.4);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.1em;color:#C6CCDD">24 HOURS</span>
            <span style="padding:12px 14px;border-radius:11px;background:rgba(4,9,26,.4);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.1em;color:#C6CCDD">MENTORS + PRIZES</span>
            <span style="padding:12px 14px;border-radius:11px;background:rgba(4,9,26,.4);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.1em;color:#C6CCDD">STATEWIDE</span>
          </div>
        </div>
      </article>
    </div>
  </section>

  <section style="position:relative;overflow:hidden;background:#060D24;padding:clamp(56px,7vw,96px) clamp(18px,3.4vw,40px);border-top:1px solid rgba(247,244,239,.08)">
    <div style="max-width:1340px;margin:0 auto;display:flex;flex-wrap:wrap;gap:24px;align-items:center;justify-content:space-between">
      <h2 data-reveal="up" style="margin:0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(24px,3.4vw,46px);line-height:1;letter-spacing:-.015em;text-transform:uppercase;max-width:16ch">Want a program in your community?</h2>
      <a data-reveal="up" data-reveal-delay="120" href="#/contact" style="display:inline-flex;align-items:center;gap:12px;padding:8px 8px 8px 22px;border-radius:999px;background:#FF6A00;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.14em;text-transform:uppercase;transition:background .3s ease,transform .3s cubic-bezier(.2,.7,.2,1)" class="scpg">Talk to us<span style="display:grid;place-items:center;width:32px;height:32px;border-radius:50%;background:#04091A;color:#FF8A2B;font-size:13px">→</span></a>
    </div>
  </section>
</div>

<div data-screen="/events" data-screen-label="Events" style="display:none">
  <section style="position:relative;overflow:hidden;background:#04091A;padding:clamp(126px,16vh,180px) clamp(18px,3.4vw,40px) clamp(48px,6vw,72px)">
    <div aria-hidden="true" style="position:absolute;inset:0;pointer-events:none;display:flex;flex-direction:column;justify-content:flex-end;gap:clamp(14px,2.4vw,30px);padding-bottom:clamp(18px,3vw,40px)">
      <span style="height:1px;background:rgba(247,244,239,.07)"></span>
      <span style="height:1px;background:rgba(247,244,239,.07)"></span>
      <span style="position:relative;height:1px;background:rgba(255,138,43,.22);overflow:hidden"><span style="position:absolute;inset:0 auto 0 0;width:18%;background:#FF6A00;animation:lhRail 7s linear infinite"></span></span>
    </div>
    <div aria-hidden="true" style="position:absolute;top:clamp(96px,13vh,158px);left:0;width:clamp(56px,7vw,120px);height:clamp(10px,1.4vw,18px);background:#FF6A00;pointer-events:none"></div>
    <div style="position:relative;max-width:1340px;margin:0 auto">
      <div style="display:inline-flex;align-items:center;gap:10px;padding:7px 14px;border-radius:999px;border:1px solid rgba(255,138,43,.32);background:rgba(255,106,0,.09);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.18em;color:#FFA65C" data-reveal="up">04 // EVENTS</div>
      <h1 data-reveal="mask" data-reveal-delay="80" style="margin:clamp(20px,2.6vw,34px) 0 0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 120;font-weight:800;font-size:clamp(40px,9vw,132px);line-height:.86;letter-spacing:-.03em;text-transform:uppercase">
        <span style="display:block">What's</span>
        <span style="display:block;color:#FF6A00">happening next</span>
      </h1>
    </div>
  </section>

  <section style="position:relative;overflow:hidden;background:#04091A;padding:0 clamp(18px,3.4vw,40px) clamp(64px,8vw,110px)">
    <div aria-hidden="true" style="position:absolute;left:clamp(6px,1.6vw,20px);top:0;bottom:0;width:2px;background:rgba(247,244,239,.08);overflow:hidden;pointer-events:none"><span style="position:absolute;inset:0 auto auto 0;width:2px;height:26%;background:#FF6A00;animation:lhBeam 9s linear infinite"></span></div>
    <div aria-hidden="true" data-grow="y" style="position:absolute;left:clamp(6px,1.6vw,20px);top:0;bottom:0;width:2px;background:#FF8A2B;transform:scaleY(0);transform-origin:top center;pointer-events:none;box-shadow:0 0 14px rgba(255,138,43,.8)"></div>
    <div style="position:relative;max-width:1340px;margin:0 auto">
      <article data-reveal="left" class="lh-erow scph" style="display:grid;grid-template-columns:minmax(0,.24fr) minmax(0,1fr) minmax(0,.26fr);gap:clamp(20px,3vw,44px);padding:clamp(26px,3vw,40px) 0;border-top:1px solid rgba(247,244,239,.16);align-items:start;transition:padding-left .4s ease">
        <div>
          <div style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 118;font-weight:800;font-size:clamp(32px,3.6vw,52px);line-height:1;letter-spacing:-.03em;text-transform:uppercase">Aug</div>
          <div style="margin-top:10px;display:inline-block;padding:6px 11px;border:1px dashed rgba(255,138,43,.6);border-radius:8px;font-family:'JetBrains Mono',monospace;font-size:11.5px;color:#FF8A2B">[DATE]</div>
        </div>
        <div>
          <div style="display:flex;flex-wrap:wrap;align-items:center;gap:10px;margin-bottom:16px">
            <span style="padding:6px 12px;border-radius:999px;background:#FF6A00;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.16em">NEXT UP</span>
            <span style="font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.18em;color:#7D87A5">WORKSHOP</span>
          </div>
          <h2 style="margin:0 0 14px;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 110;font-weight:700;font-size:clamp(22px,2.6vw,34px);line-height:1.06;letter-spacing:-.015em;text-transform:uppercase">August Community Workshop</h2>
          <p style="margin:0 0 20px;color:#9AA3BC;font-size:15.5px;line-height:1.65;max-width:50ch">A free, hands-on day of computer science for students and families. Laptops provided, no experience needed.</p>
          <div style="padding:16px 18px;border-radius:12px;border:1px dashed rgba(247,244,239,.2);background:rgba(9,16,38,.5);max-width:50ch">
            <div style="font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.2em;color:#7D87A5;margin-bottom:9px">TOPICS TAUGHT</div>
            <div style="font-family:'JetBrains Mono',monospace;font-size:13.5px;color:#FF8A2B">[placeholder — add topics]</div>
          </div>
        </div>
        <a href="#/contact" style="justify-self:start;display:inline-flex;align-items:center;gap:11px;padding:8px 8px 8px 18px;border-radius:999px;border:1px solid rgba(247,244,239,.2);color:#F7F4EF;font-family:'JetBrains Mono',monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;transition:border-color .3s ease,color .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp5">RSVP<span style="display:grid;place-items:center;width:28px;height:28px;border-radius:50%;background:rgba(247,244,239,.08);font-size:12px">→</span></a>
      </article>

      <article data-reveal="right" data-reveal-delay="120" style="position:relative;overflow:hidden;margin-top:14px;border-radius:24px;border:1px solid rgba(255,138,43,.34);background:#08122E;border-left:6px solid #FF6A00;padding:clamp(30px,4vw,60px)">
        <div aria-hidden="true" data-para="0.12" style="position:absolute;right:-1%;top:-18%;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 124;font-weight:800;font-size:clamp(90px,17vw,220px);line-height:.8;color:rgba(247,244,239,.06);pointer-events:none;user-select:none">2026</div>
        <div style="position:relative;display:flex;flex-direction:column;gap:20px;max-width:60ch">
          <div style="display:flex;flex-wrap:wrap;gap:10px">
            <span style="display:inline-flex;align-items:center;gap:8px;padding:7px 14px;border-radius:999px;background:#F7F4EF;color:#04091A;border:2px solid #04091A;font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;transform:rotate(-2.5deg)"><span style="width:6px;height:6px;border-radius:50%;background:#FF6A00;animation:lhNode 1.6s ease-in-out infinite"></span>COMING SOON</span>
            <span style="padding:7px 14px;border-radius:999px;border:1px dashed rgba(247,244,239,.34);font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.14em;color:#C6CCDD;transform:rotate(1.5deg)">DATE TBA</span>
          </div>
          <h2 style="margin:0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 120;font-weight:800;font-size:clamp(30px,5.4vw,76px);line-height:.92;letter-spacing:-.03em;text-transform:uppercase">Our first hackathon</h2>
          <p style="margin:0;color:#D7DBE6;font-size:16.5px;line-height:1.66;max-width:44ch">24 hours. Free entry. Real mentors, real prizes, and a room full of students who have never had a shot at one before.</p>
          <a href="#/contact" style="align-self:flex-start;display:inline-flex;align-items:center;gap:12px;padding:8px 8px 8px 22px;border-radius:999px;background:#F7F4EF;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.14em;text-transform:uppercase;transition:transform .3s cubic-bezier(.2,.7,.2,1)" class="scpi">Get notified<span style="display:grid;place-items:center;width:32px;height:32px;border-radius:50%;background:#FF6A00;font-size:13px">→</span></a>
        </div>
      </article>
    </div>
  </section>

  <section style="position:relative;overflow:hidden;background:#060D24;padding:clamp(56px,7vw,96px) clamp(18px,3.4vw,40px);border-top:1px solid rgba(247,244,239,.08)">
    <div style="max-width:1340px;margin:0 auto;display:grid;grid-template-columns:repeat(auto-fit,minmax(260px,1fr));gap:clamp(24px,4vw,56px);align-items:end">
      <h2 data-reveal="up" style="margin:0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:clamp(24px,3.4vw,46px);line-height:1;letter-spacing:-.015em;text-transform:uppercase;max-width:14ch">Host an event with us</h2>
      <p data-reveal="up" data-reveal-delay="100" style="margin:0;color:#9AA3BC;font-size:15.5px;line-height:1.65;max-width:42ch">Schools, libraries, and companies partner with us to bring workshops into their own communities.</p>
      <a data-reveal="up" data-reveal-delay="180" href="#/contact" style="justify-self:start;display:inline-flex;align-items:center;gap:12px;padding:8px 8px 8px 22px;border-radius:999px;background:#FF6A00;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.14em;text-transform:uppercase;transition:background .3s ease,transform .3s cubic-bezier(.2,.7,.2,1)" class="scpg">Get in touch<span style="display:grid;place-items:center;width:32px;height:32px;border-radius:50%;background:#04091A;color:#FF8A2B;font-size:13px">→</span></a>
    </div>
  </section>
</div>

<div data-screen="/contact" data-screen-label="Contact" style="display:none">
  <section style="position:relative;overflow:hidden;background:#04091A;padding:clamp(126px,16vh,180px) clamp(18px,3.4vw,40px) clamp(72px,9vw,124px)">
    <div aria-hidden="true" style="position:absolute;right:-12%;bottom:-30%;width:min(740px,80vw);aspect-ratio:1;pointer-events:none;display:grid;place-items:center">
      <div style="position:absolute;inset:0;border-radius:50%;border:1px solid rgba(255,138,43,.24)"></div>
      <div style="position:absolute;inset:0;border-radius:50%;border:1px solid #FF8A2B;animation:lhRipple 6s ease-out infinite"></div>
      <div style="position:absolute;inset:0;border-radius:50%;border:1px solid #FF8A2B;animation:lhRipple 6s 2s ease-out infinite"></div>
      <div style="position:absolute;inset:0;border-radius:50%;border:1px dashed rgba(255,138,43,.3);animation:lhSpin 70s linear infinite"></div>
    </div>
    <div aria-hidden="true" style="position:absolute;right:0;top:clamp(100px,14vh,168px);width:clamp(60px,8vw,130px);height:5px;background:#FF6A00;pointer-events:none"></div>
    <div style="position:absolute;inset:0;pointer-events:none;opacity:.35;background-image:radial-gradient(rgba(247,244,239,.14) 1px,transparent 1px);background-size:26px 26px;mask-image:radial-gradient(60% 60% at 30% 30%,#000,transparent 78%);-webkit-mask-image:radial-gradient(60% 60% at 30% 30%,#000,transparent 78%)"></div>
    <div id="lh-contactgrid" style="position:relative;max-width:1340px;margin:0 auto;display:grid;grid-template-columns:minmax(0,1fr) minmax(0,.95fr);gap:clamp(32px,5vw,80px);align-items:start">
      <div>
        <div style="display:inline-flex;align-items:center;gap:10px;padding:7px 14px;border-radius:999px;border:1px solid rgba(255,138,43,.32);background:rgba(255,106,0,.09);font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.18em;color:#FFA65C" data-reveal="up">05 // CONTACT</div>
        <h1 data-reveal="up" data-reveal-delay="80" style="margin:clamp(20px,2.6vw,34px) 0 0;font-family:Archivo,sans-serif;font-variation-settings:'wdth' 118;font-weight:800;font-size:clamp(36px,6vw,84px);line-height:.92;letter-spacing:-.025em;text-transform:uppercase;max-width:11ch">Let's build <span style="color:transparent;-webkit-text-stroke:1.4px #FF6A00">something</span> together</h1>
        <p data-reveal="up" data-reveal-delay="160" style="margin:clamp(20px,2.4vw,30px) 0 0;max-width:44ch;font-size:16.5px;line-height:1.68;color:#B9C0D4">Sponsorships, volunteering, hosting a workshop in your community, or donating hardware — tell us what you have in mind.</p>
        <div data-reveal="up" data-reveal-delay="240" style="margin-top:clamp(30px,3.6vw,46px);display:flex;flex-direction:column">
          <div style="padding:15px 0;border-top:1px solid rgba(247,244,239,.12);display:flex;justify-content:space-between;gap:16px;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.1em"><span style="color:#7D87A5">EMAIL</span><span style="color:#F7F4EF">hello@longhornhacks.org</span></div>
          <div style="padding:15px 0;border-top:1px solid rgba(247,244,239,.12);display:flex;justify-content:space-between;gap:16px;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.1em"><span style="color:#7D87A5">LOCATION</span><span style="color:#F7F4EF">AUSTIN, TEXAS</span></div>
          <div style="padding:15px 0;border-top:1px solid rgba(247,244,239,.12);border-bottom:1px solid rgba(247,244,239,.12);display:flex;justify-content:space-between;gap:16px;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.1em"><span style="color:#7D87A5">STATUS</span><span style="color:#FF8A2B">501(C)(3) NONPROFIT</span></div>
        </div>
      </div>

      <form data-reveal="right" data-reveal-delay="120" style="display:flex;flex-direction:column;gap:18px;padding:clamp(24px,3vw,38px);border-radius:22px;border:1px solid rgba(247,244,239,.12);background:rgba(9,16,38,.62);backdrop-filter:blur(16px);box-shadow:0 30px 70px rgba(0,0,0,.45)" onsubmit={onSubmit}>
        <div style="display:flex;align-items:center;justify-content:space-between;gap:12px;padding-bottom:16px;border-bottom:1px solid rgba(247,244,239,.1)">
          <span style="font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.2em;color:#7D87A5">SEND A MESSAGE</span>
          <span style="display:flex;gap:5px"><span style="width:7px;height:7px;border-radius:50%;background:#FF6A00"></span><span style="width:7px;height:7px;border-radius:50%;background:rgba(247,244,239,.2)"></span><span style="width:7px;height:7px;border-radius:50%;background:rgba(247,244,239,.2)"></span></span>
        </div>
        <label style="display:flex;flex-direction:column;gap:9px">
          <span style="font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.2em;color:#7D87A5">NAME</span>
          <input type="text" placeholder="Your name" style="padding:14px 16px;border-radius:11px;border:1px solid rgba(247,244,239,.14);background:rgba(4,9,26,.5);color:#F7F4EF;font-size:15.5px;outline:none;transition:border-color .3s ease,box-shadow .3s ease"  class="scpj"/>
        </label>
        <label style="display:flex;flex-direction:column;gap:9px">
          <span style="font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.2em;color:#7D87A5">EMAIL</span>
          <input type="email" placeholder="you@company.com" style="padding:14px 16px;border-radius:11px;border:1px solid rgba(247,244,239,.14);background:rgba(4,9,26,.5);color:#F7F4EF;font-size:15.5px;outline:none;transition:border-color .3s ease,box-shadow .3s ease"  class="scpj"/>
        </label>
        <label style="display:flex;flex-direction:column;gap:9px">
          <span style="font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.2em;color:#7D87A5">MESSAGE</span>
          <textarea rows="4" placeholder="How would you like to get involved?" style="padding:14px 16px;border-radius:11px;border:1px solid rgba(247,244,239,.14);background:rgba(4,9,26,.5);color:#F7F4EF;font-size:15.5px;outline:none;resize:vertical;transition:border-color .3s ease,box-shadow .3s ease" class="scpj"></textarea>
        </label>
        <button type="submit" style="display:inline-flex;align-items:center;justify-content:space-between;gap:12px;margin-top:4px;padding:8px 8px 8px 22px;border-radius:999px;border:none;background:#FF6A00;color:#04091A;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.14em;text-transform:uppercase;cursor:pointer;transition:background .3s ease,transform .3s cubic-bezier(.2,.7,.2,1)" class="scpk">{submitLabel}<span style="display:grid;place-items:center;width:32px;height:32px;border-radius:50%;background:#04091A;color:#FF8A2B;font-size:13px">→</span></button>
      </form>
    </div>
  </section>
</div>

</main>

<footer style="position:relative;overflow:hidden;background:#04091A;padding:clamp(52px,6.4vw,84px) clamp(18px,3.4vw,40px) 28px;border-top:1px solid rgba(247,244,239,.1)">
  <div style="max-width:1340px;margin:0 auto">
    <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:clamp(26px,4vw,56px);align-items:start">
      <div>
        <div style="display:flex;align-items:center;gap:12px">
          <span style="display:grid;place-items:center;width:46px;height:46px;border-radius:50%;background:#F7F4EF">
            <img src={lhhLogo} alt="Longhorn Hacks logo" width="36" height="36" style="display:block;width:36px;height:36px;object-fit:contain" />
          </span>
          <span style="font-family:Archivo,sans-serif;font-variation-settings:'wdth' 112;font-weight:700;font-size:16px;letter-spacing:.02em;text-transform:uppercase">Longhorn Hacks</span>
        </div>
        <p style="margin:18px 0 0;color:#7D87A5;font-size:14px;line-height:1.62;max-width:30ch">Expanding access to computer science and AI education across Texas.</p>
      </div>
      <div style="display:flex;flex-direction:column;gap:12px">
        <span style="font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.2em;color:#5E688A;margin-bottom:4px">SITE</span>
        <a href="#/" style="color:#A8B0C8;font-size:14px;transition:color .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp4">Home</a>
        <a href="#/about" style="color:#A8B0C8;font-size:14px;transition:color .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp4">About</a>
        <a href="#/programs" style="color:#A8B0C8;font-size:14px;transition:color .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp4">Programs</a>
      </div>
      <div style="display:flex;flex-direction:column;gap:12px">
        <span style="font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.2em;color:#5E688A;margin-bottom:4px">GET INVOLVED</span>
        <a href="#/events" style="color:#A8B0C8;font-size:14px;transition:color .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp4">Events</a>
        <a href="#/contact" style="color:#A8B0C8;font-size:14px;transition:color .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp4">Contact</a>
        <a href="#/contact" style="color:#A8B0C8;font-size:14px;transition:color .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scp4">Donate</a>
      </div>
      <div>
        <span style="display:block;font-family:'JetBrains Mono',monospace;font-size:9.5px;letter-spacing:.2em;color:#5E688A;margin-bottom:16px">LEGAL</span>
        <p style="margin:0;color:#7D87A5;font-size:13px;line-height:1.65;max-width:34ch">Longhorn Hacks is a registered 501(c)(3) nonprofit organization. Donations are tax-deductible to the extent allowed by law.</p>
        <p style="margin:11px 0 0;font-family:'JetBrains Mono',monospace;font-size:10.5px;color:#5E688A">EIN [PLACEHOLDER]</p>
      </div>
    </div>
    <div aria-hidden="true" data-para="0.07" style="margin-top:clamp(40px,5vw,72px);font-family:Archivo,sans-serif;font-variation-settings:'wdth' 124;font-weight:800;font-size:clamp(40px,12.6vw,190px);letter-spacing:-.035em;line-height:.82;color:rgba(247,244,239,.06);white-space:nowrap;user-select:none;text-transform:uppercase">Longhorn Hacks</div>
    <div style="margin-top:clamp(22px,2.6vw,36px);padding-top:18px;border-top:1px solid rgba(247,244,239,.1);display:flex;flex-wrap:wrap;gap:14px;align-items:center;justify-content:space-between">
      <span style="font-family:'JetBrains Mono',monospace;font-size:10.5px;letter-spacing:.1em;color:#5E688A">© 2026 LONGHORN HACKS — ALL RIGHTS RESERVED</span>
      <button type="button" onclick={onTop} style="display:inline-flex;align-items:center;gap:12px;padding:8px 20px 8px 8px;border-radius:999px;border:1px solid rgba(255,138,43,.5);background:rgba(255,106,0,.12);cursor:pointer;font-family:'JetBrains Mono',monospace;font-size:11.5px;letter-spacing:.16em;color:#FFA65C;transition:background .3s ease,border-color .3s ease,color .3s ease,transform .35s cubic-bezier(.2,.7,.2,1)" class="scpl"><span style="display:grid;place-items:center;width:34px;height:34px;border-radius:50%;background:#FF6A00;color:#04091A;font-size:14px">↑</span>BACK TO TOP</button>
    </div>
  </div>
</footer>

</div>
