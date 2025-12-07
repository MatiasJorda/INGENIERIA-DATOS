---
title: "Indice del Portafolio"
date: 2025-01-01
---

# Portafolio

Este espacio reune las practicas y los resumenes de cada unidad de la materia. Usa los accesos rapidos para saltar a las practicas o a los resumenes.

<style>
.sections-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 1.25rem;
  margin: 1.5rem 0 2.5rem 0;
}
.section-card {
  background: var(--md-default-bg-color);
  border: 1px solid var(--md-default-fg-color--lighter);
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  text-decoration: none;
  color: inherit;
  display: block;
  position: relative;
  overflow: hidden;
  transition: all 0.25s ease;
}
.section-card::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, var(--md-primary-fg-color), var(--md-primary-fg-color--dark));
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 0.25s ease;
}
.section-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 18px rgba(0, 0, 0, 0.12);
  border-color: var(--md-primary-fg-color);
}
.section-card:hover::before {
  transform: scaleX(1);
}
.section-card h3 {
  margin: 0 0 0.4rem 0;
  font-weight: 600;
  background: linear-gradient(90deg, var(--md-primary-fg-color), var(--md-primary-fg-color--dark));
  -webkit-background-clip: text;
  color: transparent;
}
.section-card p {
  margin: 0 0 0.6rem 0;
  color: var(--md-default-fg-color--light);
}
.section-card .section-link {
  display: inline-flex;
  align-items: center;
  gap: 0.35rem;
  color: var(--md-primary-fg-color);
  font-weight: 600;
  text-decoration: none;
}
.section-card:hover .section-link svg {
  transform: translateX(4px);
}
.section-card .section-link svg {
  width: 14px;
  height: 14px;
  transition: transform 0.2s ease;
}
@media (max-width: 768px) {
  .sections-grid {
    grid-template-columns: 1fr;
  }
}
</style>

<div class="sections-grid">
  <a class="section-card" href="Practicas_unidad">
    <h3>Practicas</h3>
    <p>Ejercicios y proyectos practicos por unidad: EDA, calidad de datos, feature engineering, datos especiales y MLOps.</p>
    <span class="section-link">Explorar practicas
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"></path><path d="m13 5 7 7-7 7"></path></svg>
    </span>
  </a>
  <a class="section-card" href="Resumenes_unidad">
    <h3>Resumenes</h3>
    <p>Apuntes y reflexiones por unidad con los conceptos principales y enlaces a notebooks de soporte.</p>
    <span class="section-link">Ver resumenes
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"></path><path d="m13 5 7 7-7 7"></path></svg>
    </span>
  </a>
</div>
