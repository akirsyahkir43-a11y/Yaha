/* Y-VERS — Supabase connection */

const SUPABASE_URL = "https://ekdgxwjjfgoltgfocfyv.supabase.co";
const SUPABASE_KEY = "sb_publishable_lef_siYnSBIvo_6jsS7OQw_syHwP0gh";

let supabaseClient;
let currentUser = null;
let currentPage = "home";

function toast(message) {
  const el = document.getElementById("toast");
  if (!el) return;
  el.textContent = message;
  el.classList.add("show");
  clearTimeout(window.__yversTimer);
  window.__yversTimer = setTimeout(
    () => el.classList.remove("show"), 2800
  );
}

window.toast = toast;

function escapeHTML(value) {
  return String(value ?? "").replace(/[&<>"']/g, c => ({
    "&": "&amp;",
    "<": "&lt;",
    ">": "&gt;",
    '"': "&quot;",
    "'": "&#39;"
  })[c]);
}

function initYVers() {
  if (!window.supabase) {
    toast("Supabase gagal dimuat.");
    return;
  }

  supabaseClient = window.supabase.createClient(
    SUPABASE_URL,
    SUPABASE_KEY
  );

  supabaseClient.auth.getSession().then(({ data }) => {
    currentUser = data.session?.user ?? null;
    renderPage(currentPage);
  });

  supabaseClient.auth.onAuthStateChange((_event, session) => {
    currentUser = session?.user ?? null;
    renderPage(currentPage);
  });

  document.querySelectorAll("[data-page]").forEach(button => {
    button.addEventListener("click", () => {
      currentPage = button.dataset.page;

      document.querySelectorAll("[data-page]").forEach(item => {
        item.classList.toggle("active", item === button);
      });

      renderPage(currentPage);
    });
  });

  renderPage("home");
}

function shell(title, body) {
  return `
    <section class="page">
      <h1>${escapeHTML(title)}</h1>
      ${body}
    </section>
  `;
}

function authPanel() {
  return `
    <div class="auth-card">
      <h2>Masuk ke Y-VERS</h2>

      <form id="login-form">
        <input name="email" type="email"
          placeholder="Email" required>

        <input name="password" type="password"
          placeholder="Password" minlength="6" required>

        <button type="submit">Log In</button>
      </form>

      <button id="signup-button" type="button">
        Create Account
      </button>
    </div>
  `;
}

function renderPage(page) {
  const screen = document.getElementById("screen");
  if (!screen) return;

  if (page === "profile" && !currentUser) {
    screen.innerHTML = shell("My / Akun", authPanel());
    wireAuth();
    return;
  }

  if (page === "home") {
    screen.innerHTML = shell("Y-VERS", `
      <div class="hero-card">
        <h2>Y-VERS</h2>
        <p>K-POP · ENTERTAINMENT · BEYOND</p>
      </div>
    `);
  }

  else if (page === "program") {
    const programs = [
      "GAH", "V:LAB", "NXT GENERATION",
      "V-SYNC", "ID:ONE", "V-MODE",
      "V-LOGUE", "UN:ITY", "V-CLUB", "V-CHART"
    ];

    screen.innerHTML = shell("Program", `
      <div class="program-grid">
        ${programs.map(name =>
          `<div class="section-card"><h3>${name}</h3></div>`
        ).join("")}
      </div>
    `);
  }

  else if (page === "gah") {
    screen.innerHTML = shell("GAH Awards", `
      <div class="section-card">
        <h3>Voting GAH</h3>
        <p>Kategori dan nominasi akan ditampilkan di sini.</p>
      </div>
    `);
  }

  else if (page === "community") {
    screen.innerHTML = shell("Community", `
      <div class="section-card">
        <h3>Y-VERS Community</h3>
        ${currentUser
          ? `<p>Login sebagai ${escapeHTML(currentUser.email)}</p>`
          : authPanel()}
      </div>
    `);

    if (!currentUser) wireAuth();
  }

  else if (page === "profile") {
    screen.innerHTML = shell("My", `
      <div class="section-card">
        <p>${escapeHTML(currentUser.email)}</p>
        <button id="logout-button">Log Out</button>
      </div>
    `);

    document.getElementById("logout-button")
      ?.addEventListener("click", async () => {
        const { error } = await supabaseClient.auth.signOut();
        if (error) toast(error.message);
        else toast("Berhasil keluar.");
      });
  }
}

function wireAuth() {
  document.getElementById("login-form")
    ?.addEventListener("submit", async event => {
      event.preventDefault();

      const form = new FormData(event.currentTarget);

      const { error } =
        await supabaseClient.auth.signInWithPassword({
          email: form.get("email"),
          password: form.get("password")
        });

      if (error) toast("Login gagal: " + error.message);
      else toast("Login berhasil.");
    });

  document.getElementById("signup-button")
    ?.addEventListener("click", async () => {
      const form = document.getElementById("login-form");

      if (!form.reportValidity()) return;

      const { error } = await supabaseClient.auth.signUp({
        email: form.elements.email.value.trim(),
        password: form.elements.password.value
      });

      if (error) toast("Pendaftaran gagal: " + error.message);
      else toast("Pendaftaran berhasil dikirim. Periksa email.");
    });
}

const supabaseScript = document.createElement("script");
supabaseScript.src =
  "https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2";

supabaseScript.onload = initYVers;
supabaseScript.onerror = () =>
  toast("Gagal memuat Supabase.");

document.head.appendChild(supabaseScript);
