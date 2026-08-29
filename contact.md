---
layout: default
title: Contact
---

<div id="contact">
  <h1 class="pageTitle">Contact Me</h1>
  <div class="contactContent">
    <p class="intro">Feel free to reach out to me! I'm always open to discussing new opportunities, collaborations, or just having a chat about tech.</p>
    
    <div class="contact-email">
        <p>You can drop me an email at: <br>
        <strong><a href="mailto:{{ site.social.email }}">{{ site.social.email }}</a></strong></p>
    </div>

    <div class="social-grid">
      {% if site.social.github %}
      <a href="https://github.com/{{ site.social.github }}" target="_blank" class="social-item">
        <svg id="github" class="custom-icon" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 100 100">
            <circle class="outer-shape" cx="50" cy="50" r="48" style="opacity: 1; fill: #444;"></circle>
            <path class="inner-shape" style="opacity: 1; fill: white;" transform="translate(25,25) scale(0.5)" d="M50,1C22.938,1,1,22.938,1,50s21.938,49,49,49s49-21.938,49-49S77.062,1,50,1z M79.099,79.099 c-3.782,3.782-8.184,6.75-13.083,8.823c-1.245,0.526-2.509,0.989-3.79,1.387v-7.344c0-3.86-1.324-6.699-3.972-8.517 c1.659-0.16,3.182-0.383,4.57-0.67c1.388-0.287,2.855-0.702,4.402-1.245c1.547-0.543,2.935-1.189,4.163-1.938 c1.228-0.75,2.409-1.723,3.541-2.919s2.082-2.552,2.847-4.067s1.372-3.334,1.818-5.455c0.446-2.121,0.67-4.458,0.67-7.01 c0-4.945-1.611-9.155-4.833-12.633c1.467-3.828,1.308-7.991-0.478-12.489l-1.197-0.143c-0.829-0.096-2.321,0.255-4.474,1.053 c-2.153,0.798-4.57,2.105-7.249,3.924c-3.797-1.053-7.736-1.579-11.82-1.579c-4.115,0-8.039,0.526-11.772,1.579 c-1.69-1.149-3.294-2.097-4.809-2.847c-1.515-0.75-2.727-1.26-3.637-1.532c-0.909-0.271-1.754-0.439-2.536-0.503 c-0.782-0.064-1.284-0.079-1.507-0.048c-0.223,0.031-0.383,0.064-0.478,0.096c-1.787,4.53-1.946,8.694-0.478,12.489 c-3.222,3.477-4.833,7.688-4.833,12.633c0,2.552,0.223,4.889,0.67,7.01c0.447,2.121,1.053,3.94,1.818,5.455 c0.765,1.515,1.715,2.871,2.847,4.067s2.313,2.169,3.541,2.919c1.228,0.751,2.616,1.396,4.163,1.938 c1.547,0.543,3.014,0.957,4.402,1.245c1.388,0.287,2.911,0.511,4.57,0.67c-2.616,1.787-3.924,4.626-3.924,8.517v7.487 c-1.445-0.43-2.869-0.938-4.268-1.53c-4.899-2.073-9.301-5.041-13.083-8.823c-3.782-3.782-6.75-8.184-8.823-13.083 C9.934,60.948,8.847,55.56,8.847,50s1.087-10.948,3.231-16.016c2.073-4.899,5.041-9.301,8.823-13.083s8.184-6.75,13.083-8.823 C39.052,9.934,44.44,8.847,50,8.847s10.948,1.087,16.016,3.231c4.9,2.073,9.301,5.041,13.083,8.823 c3.782,3.782,6.75,8.184,8.823,13.083c2.143,5.069,3.23,10.457,3.23,16.016s-1.087,10.948-3.231,16.016 C85.848,70.915,82.88,75.317,79.099,79.099L79.099,79.099z" />
        </svg>
        <span class="social-name">GitHub</span>
        <span class="social-username">@{{ site.social.github }}</span>
      </a>
      {% endif %}

      {% if site.social.linkedin %}
      <a href="https://www.linkedin.com/in/{{ site.social.linkedin }}" target="_blank" class="social-item">
        <svg id="linkedin" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 96 96">
            <circle class="outer-shape" cx="48" cy="48" r="48" fill="#0077b5" />
            <path class="inner-shape" fill="#FFF" d="M10.2966071,46 L0.759821429,46 L0.759821429,15.2881537 L10.2966071,15.2881537 L10.2966071,46 Z M5.52308036,11.0987745 C2.47352679,11.0987745 1.17239551e-12,8.57282522 1.17239551e-12,5.52320357 C1.17239551e-12,2.47282245 2.47276734,-9.65449942e-13 5.52308036,-9.65449942e-13 C8.57339338,-9.65449942e-13 11.0461607,2.47282245 11.0461607,5.52320357 C11.0461607,8.57282522 8.57160714,11.0987745 5.52308036,11.0987745 Z M45.9897321,46 L36.4734821,46 L36.4734821,31.0496663 C36.4734821,27.4866403 36.4016071,22.9173419 31.5151339,22.9173419 C26.5567857,22.9173419 25.7969643,26.7884104 25.7969643,30.7929641 L25.7969643,46 L16.2704464,46 L16.2704464,15.2881537 L25.4170536,15.2881537 L25.4170536,19.4775329 L25.5505357,19.4775329 C26.82375,17.0645326 29.9338839,14.5180472 34.5739286,14.5180472 C44.2257143,14.5180472 46,20.8739927 46,29.1295341 L46,46 L45.9897321,46 Z" transform="translate(25 25)" />
        </svg>
        <span class="social-name">LinkedIn</span>
        <span class="social-username">{{ site.social.linkedin }}</span>
      </a>
      {% endif %}



      {% if site.social.discord %}
      <a href="https://discord.com/users/{{ site.social.discord }}" target="_blank" class="social-item">
        <svg id="discord" class="custom-icon" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 100 100">
            <circle class="outer-shape" cx="50" cy="50" r="48" style="opacity: 1; fill: #5865f2;"></circle>
            <path class="inner-shape" style="opacity: 1; fill: white;" transform="translate(25,25) scale(0.5)" d="M73.03,31.01c-4.87-4.51-11.53-6.93-17.97-6.93l-0.89,1.1c7.46,2.21,11.05,5.55,11.05,5.55c-4.62-2.52-9.14-3.76-13.43-4.39 c-3.11-0.45-6.19-0.57-9.2-0.43c-6.11,0.29-11.69,1.48-16.71,3.4c-2.73,1.05-4.4,1.88-4.4,1.88s3.76-3.53,11.75-5.77l-0.78-1.01 c-6.44,0-13.1,2.42-17.97,6.93c-1.39,11.83-4.63,26.11-0.29,36.56c4.95,5.59,13.78,6.01,13.78,6.01l2.52-3.3 c-4.64-1.33-8.81-4.06-8.81-4.06c0.55,0.36,1.04,0.66,1.54,0.92c4.11,2.15,8.19,3.53,12.71,4.4c3.04,0.59,6.59,0.91,10.15,0.91 c3.56,0,7.11-0.32,10.15-0.91c4.52-0.87,8.6-2.25,12.71-4.4c0.5-0.26,0.99-0.56,1.54-0.92c0,0-4.17,2.73-8.81,4.06l2.52,3.3 c0,0,8.83-0.42,13.78-6.01C77.66,57.12,74.42,42.84,73.03,31.01z M40.75,56.63c-3.11,0-5.63-2.73-5.63-6.09 c0-3.36,2.52-6.09,5.63-6.09c3.11,0,5.63,2.73,5.63,6.09C46.38,53.9,43.86,56.63,40.75,56.63z M59.25,56.63 c-3.11,0-5.63-2.73-5.63-6.09c0-3.36,2.52-6.09,5.63-6.09s5.63,2.73,5.63,6.09C64.88,53.9,62.36,56.63,59.25,56.63z"></path>
        </svg>
        <span class="social-name">Discord</span>
        <span class="social-username">{{ site.social.discord }}</span>
      </a>
      {% endif %}

      {% if site.social.discourse %}
      <a href="https://octave.discourse.group/u/{{ site.social.discourse }}" target="_blank" class="social-item">
        <svg id="discourse" class="custom-icon" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 100 100">
            <circle class="outer-shape" cx="50" cy="50" r="48" style="opacity: 1; fill: #222;"></circle>
            <path class="inner-shape" style="opacity: 1; fill: white;" transform="translate(25,25) scale(0.5)" d="M50,1C22.9,1,1,22.9,1,50s21.9,49,49,49c2.1,0,4.2-0.1,6.2-0.4l15,7.5l-0.8-15.6C89,79.5,99,65.9,99,50C99,22.9,77.1,1,50,1z M35,65 c-3.3,0-6-2.7-6-6s2.7-6,6-6s6,2.7,6,6S38.3,65,35,65z M35,47c-3.3,0-6-2.7-6-6s2.7-6,6-6s6,2.7,6,6S38.3,47,35,47z M65,65 c-3.3,0-6-2.7-6-6s2.7-6,6-6s6,2.7,6,6S68.3,65,65,65z M65,47c-3.3,0-6-2.7-6-6s2.7-6,6-6s6,2.7,6,6S68.3,47,65,47z"></path>
        </svg>
        <span class="social-name">Discourse</span>
        <span class="social-username">Avanish</span>
      </a>
      {% endif %}
    </div>
  </div>
</div>

<style>
#contact {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  float: none !important; /* Overriding theme's 800up float */
}

.pageTitle {
  text-align: center !important;
  margin-bottom: 2rem;
}

.contactContent {
  width: 100% !important;
  max-width: 100% !important;
  float: none !important; /* Overriding theme's 800up float */
  margin: 0 auto !important;
  text-align: center;
}

.intro {
  text-align: left;
  max-width: 700px; /* Wider to allow spanning into 2-3 lines nicely */
  margin: 0 auto 2rem;
  line-height: 1.6;
}

.contact-email {
  margin: 2rem auto 3rem;
  padding: 1.5rem;
  background: rgba(0,0,0,0.03);
  border-radius: 12px;
  font-size: 1.1rem;
  text-align: center;
  width: 100%;
}

.social-grid {
  display: flex;
  justify-content: center;
  gap: 3.5rem;
  flex-wrap: nowrap;
  width: 100%;
  margin: 3rem auto;
}

.social-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-decoration: none !important;
  color: inherit !important;
}

.social-item:hover {
  transform: translateY(-5px);
}

.social-item svg {
  width: 60px;
  height: 60px;
  margin-bottom: 0.75rem;
}

.social-name {
  font-weight: 800;
  display: block;
  font-family: "Open Sans", sans-serif;
  text-transform: uppercase;
  font-size: 0.85rem;
  letter-spacing: 1px;
}

.social-username {
  font-size: 0.9rem;
  opacity: 0.7;
}

.social-note {
  font-size: 0.75rem;
  opacity: 0.6;
  font-style: italic;
  margin-top: 2px;
}

@media (prefers-color-scheme: dark) {
  .contact-email {
    background: rgba(255,255,255,0.05);
  }
}
</style>
