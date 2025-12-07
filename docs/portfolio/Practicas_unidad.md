# Practicas por unidad

Accesos rapidos a las practicas de cada unidad. Cada tarjeta es clickeable.

<style>
.sections-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.1rem;
  margin: 1.4rem 0 2rem 0;
}
.section-card {
  background: var(--md-default-bg-color);
  border: 1px solid var(--md-default-fg-color--lighter);
  border-radius: 12px;
  padding: 1.4rem;
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
  margin: 0 0 0.35rem 0;
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
  <a class="section-card" href="../UT1/01-Main/">
    <h3>UT1 · EDA y Fundamentos</h3>
    <p>Setup de repos, EDA inicial y exploracion Iris, Netflix y NYC Taxi.</p>
    <span class="section-link">Ver practicas
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"></path><path d="m13 5 7 7-7 7"></path></svg>
    </span>
  </a>
  <a class="section-card" href="../UT2/02-Main/">
    <h3>UT2 · Calidad y Etica</h3>
    <p>Missing data, escalado sin leakage y fairness con Fairlearn.</p>
    <span class="section-link">Ver practicas
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"></path><path d="m13 5 7 7-7 7"></path></svg>
    </span>
  </a>
  <a class="section-card" href="../UT3/03-Main/">
    <h3>UT3 · Feature Engineering</h3>
    <p>Encoding avanzado, PCA, selection y features temporales.</p>
    <span class="section-link">Ver practicas
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"></path><path d="m13 5 7 7-7 7"></path></svg>
    </span>
  </a>
  <a class="section-card" href="../UT4/04-Main/">
    <h3>UT4 · Datos especiales</h3>
    <p>Geoespacial con GeoPandas, preprocesamiento de imagen y audio.</p>
    <span class="section-link">Ver practicas
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"></path><path d="m13 5 7 7-7 7"></path></svg>
    </span>
  </a>
  <a class="section-card" href="../UT5/05-Main/">
    <h3>UT5 · ML Ops</h3>
    <p>Labs de Google Cloud, Dataprep y orquestacion ETL con Prefect.</p>
    <span class="section-link">Ver practicas
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"></path><path d="m13 5 7 7-7 7"></path></svg>
    </span>
  </a>
</div>

## Lista rapida
- UT1: EDA & Fundamentos - [ver practicas](UT1/01-Main.md)
- UT2: Calidad & Etica - [ver practicas](UT2/02-Main.md)
- UT3: Feature Engineering - [ver practicas](UT3/03-Main.md)
- UT4: Datos especiales - [ver practicas](UT4/04-Main.md)
- UT5: ML Ops - [ver practicas](UT5/05-Main.md)
