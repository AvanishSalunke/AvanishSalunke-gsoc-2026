---
layout: default
title: About Avanish Salunke
---

<div id="about">
  <h1 class="pageTitle">About Me</h1>

  <div class="about-content">
    <div class="profile-container">
      <img src="{{ '/assets/img/profile.jpg' | relative_url }}" alt="Avanish Salunke" class="profile-img">
    </div>
    <div class="about-box bio-box">
      <p>I’m <strong>Avanish Salunke</strong>, a B.Tech IT student at VJTI, Mumbai, working at the intersection of mathematics and computing. I’m a Google Summer of Code 2026 contributor with GNU Octave, where my project focuses on adding LinearModel and CompactLinearModel objects to the statistics package. This involves designing and implementing structured, MATLAB-like model classes for regression, handling fitting, parameter estimation, and efficient prediction workflows within Octave’s ecosystem. The work focuses on building clean, consistent abstractions that integrate well with the existing codebase and improve usability for statistical modeling. This blog is where I’ll share my weekly progress, what I’m building, and the challenges I run into along the way.</p>
    </div>

    <div class="about-box project-box">
      <div class="project-header">
        <img src="https://upload.wikimedia.org/wikipedia/commons/0/08/GSoC_logo.svg" alt="GSoC Logo" class="gsoc-logo">
        <h2 class="section-heading">Google Summer of Code 2026</h2>
      </div>
      <div class="project-details">
        <p><strong>Project Title:</strong> Add LinearModel and CompactLinearModel objects to the statistics package</p>
        <p><a href="https://summerofcode.withgoogle.com/programs/2026/projects/YKFoacXh" target="_blank">Click here to see my project</a></p>
        
        <p>This project involves implementing a robust object-oriented framework for linear regression within the GNU Octave Statistics package. By introducing the <code>LinearModel</code> and <code>CompactLinearModel</code> classes, we aim to provide deep statistical diagnostics and memory-efficient deployment options, bringing Octave closer to MATLAB compatibility.</p>
      </div>
    </div>

    <div class="about-box org-box">
      <div class="org-header">
        <img src="https://upload.wikimedia.org/wikipedia/commons/6/6a/Gnu-octave-logo.svg" alt="GNU Octave Logo" class="octave-logo">
        <h2 class="section-heading">Organization & Mentors</h2>
      </div>
      
      <div class="mentors-grid">
        <div class="mentor-item">
          <span class="label">Organization</span>
          <a href="https://www.gnu.org/software/octave/" target="_blank">GNU Octave</a>
        </div>
        <div class="mentor-item">
          <span class="label">Mentor</span>
          <a href="https://octave.discourse.group/u/pr0m1th3as/summary" target="_blank">Andreas Bertsatos</a>
        </div>
        <div class="mentor-item">
          <span class="label">Mentor</span>
          <a href="https://octave.discourse.group/u/cbm/summary" target="_blank">Colin B. Macdonald</a>
        </div>
      </div>
    </div>
  </div>
</div>

<style>
#about {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  float: none !important;
}

.pageTitle {
  text-align: center !important;
  margin-bottom: 3rem;
}

.about-content {
  width: 100% !important;
  max-width: 100% !important;
  float: none !important;
  margin: 0 auto !important;
}

.about-box {
  margin-bottom: 2.5rem;
  padding: 2rem;
  background: rgba(0,0,0,0.03);
  border-radius: 12px;
  line-height: 1.7;
}

.bio-box {
  background: none;
  padding: 0 1rem;
  text-align: justify;
}

.profile-container {
  display: flex;
  justify-content: center;
  margin-top: -6px;
  margin-bottom: 2rem;
}

.profile-img {
  width: 180px;
  height: 180px;
  border-radius: 50%;
  object-fit: cover;
  object-position: center center;
  border: 4px solid #FFDC00;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
  transition: transform 0.3s ease;
}

.profile-img:hover {
  transform: scale(1.05);
}

.project-details {
  text-align: justify;
}

.section-heading {
  margin-top: 0;
  margin-bottom: 0;
  font-size: 1.4rem;
  font-weight: 700;
}

.project-header, .org-header {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  margin-bottom: 1.5rem;
  border-bottom: 1px solid rgba(0,0,0,0.05);
  padding-bottom: 0.75rem;
}

.gsoc-logo, .octave-logo {
  width: 50px;
  height: 50px;
  object-fit: contain;
}

.mentors-grid {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  margin-top: 1.5rem;
  flex-wrap: wrap;
}

.mentor-item {
  flex: 1;
  min-width: 180px;
  background: rgba(255,255,255,0.6);
  padding: 1.2rem;
  border-radius: 8px;
  text-align: center;
}

.mentor-item .label {
  display: block;
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 1.2px;
  opacity: 0.6;
  margin-bottom: 0.4rem;
}

.mentor-item a {
  font-weight: 700;
  text-decoration: none;
  color: #0080C0;
}

@media (prefers-color-scheme: dark) {
  .about-box {
    background: rgba(255,255,255,0.05);
  }
  .mentor-item {
    background: rgba(0,0,0,0.3);
  }
  .project-header, .org-header {
    border-bottom-color: rgba(255,255,255,0.1);
  }
}

@media (max-width: 600px) {
  .project-header, .org-header {
    flex-direction: column;
    text-align: center;
  }
  .mentor-item {
    min-width: 100%;
  }
  .bio-box, .project-details {
    text-align: left; /* Better for mobile */
  }
}
</style>
