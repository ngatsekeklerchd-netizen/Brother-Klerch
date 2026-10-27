# Brother-Klerch
site web de Carriere professionnelle 
// ====== TES DONNÉES : modifie ces listes ======

// Sorties : laisse la liste vide tant que tu n'as rien à afficher.
// Exemple : { title: "Nom du titre", type: "Single", year: 2026, link: "https://audiomack.com/...", colors: ["#e8b04a", "#7a4a12"] },
const releases = [];

// Concerts : date au format AAAA-MM-JJ. Laisse vide s'il n'y en a pas.
// Exemple : { date: "2026-12-05", city: "Brazzaville", venue: "Nom de l'église ou de la salle", ticket: "#", soldOut: false },
const events = [];

const CONTACT_EMAIL = "ngatsekeklerchd@gmail.com";

// Pour recevoir les messages directement (sans ouvrir l'application mail du visiteur),
// crée un formulaire gratuit sur formspree.io et colle ici son adresse
// (ex. "https://formspree.io/f/xxxxxxx"). Sinon, laisse vide : le formulaire ouvrira l'email.
const FORM_ENDPOINT = "";

// ====== CODE DU SITE ======

const monthFmt = new Intl.DateTimeFormat("fr-FR", { month: "short" });
const yearFmt = new Intl.DateTimeFormat("fr-FR", { year: "numeric" });

function escapeHTML(str) {
  const div = document.createElement("div");
  div.textContent = str;
  return div.innerHTML;
}

// Sorties
const releasesEl = document.getElementById("releases");
if (releases.length) {
  releasesEl.innerHTML = releases.map(r => `
    <article class="card">
      <div class="card-cover" style="background: linear-gradient(135deg, ${r.colors[0]}, ${r.colors[1]})">
        ${escapeHTML(r.title)}
      </div>
      <div class="card-body">
        <h3>${escapeHTML(r.title)}</h3>
        <p>${escapeHTML(r.type)} · ${r.year}</p>
        <a href="${encodeURI(r.link)}" target="_blank" rel="noopener">Écouter →</a>
      </div>
    </article>
  `).join("");
} else {
  releasesEl.style.display = "none";
}

// Concerts (seuls les concerts à venir sont affichés)
const today = new Date();
today.setHours(0, 0, 0, 0);
const upcoming = events
  .filter(e => new Date(e.date) >= today)
  .sort((a, b) => new Date(a.date) - new Date(b.date));

document.getElementById("events").innerHTML = upcoming.length
  ? upcoming.map(e => {
      const d = new Date(e.date);
      return `
        <li class="event">
          <div class="event-date">${d.getDate()} ${monthFmt.format(d).toUpperCase()}<small>${yearFmt.format(d)}</small></div>
          <div class="event-info">
            <h3>${escapeHTML(e.city)}</h3>
            <p>${escapeHTML(e.venue)}</p>
          </div>
          ${e.soldOut
            ? '<span class="sold-out">Complet</span>'
            : `<a class="btn btn-primary" href="${encodeURI(e.ticket)}" target="_blank" rel="noopener">Billets</a>`}
        </li>`;
    }).join("")
  : '<li class="event"><p>Pas de concert prévu pour le moment. Reste connecté !</p></li>';

// Navigation : fond au scroll + menu mobile
const nav = document.getElementById("nav");
const burger = document.getElementById("burger");
const menu = document.getElementById("menu");

window.addEventListener("scroll", () => {
  nav.classList.toggle("scrolled", window.scrollY > 40);
}, { passive: true });

burger.addEventListener("click", () => {
  const open = menu.classList.toggle("open");
  burger.setAttribute("aria-expanded", open);
});
menu.querySelectorAll("a").forEach(a => a.addEventListener("click", () => {
  menu.classList.remove("open");
  burger.setAttribute("aria-expanded", "false");
}));

// Formulaire de contact
const form = document.getElementById("contact-form");
const formStatus = document.getElementById("form-status");

form.addEventListener("submit", async (e) => {
  e.preventDefault();
  formStatus.className = "form-status";

  if (!form.checkValidity()) {
    formStatus.textContent = "Merci de remplir tous les champs correctement.";
    formStatus.classList.add("error");
    return;
  }

  const data = new FormData(form);

  // Sans Formspree : on ouvre l'application mail avec le message déjà rédigé
  if (!FORM_ENDPOINT) {
    const subject = encodeURIComponent("Message depuis le site de " + data.get("nom"));
    const body = encodeURIComponent(
      data.get("message") + "\n\n" + data.get("nom") + " (" + data.get("email") + ")"
    );
    window.location.href = "mailto:" + CONTACT_EMAIL + "?subject=" + subject + "&body=" + body;
    formStatus.textContent = "Ton application mail va s'ouvrir : il te suffira d'appuyer sur Envoyer.";
    formStatus.classList.add("ok");
    return;
  }

  try {
    const res = await fetch(FORM_ENDPOINT, {
      method: "POST",
      headers: { Accept: "application/json" },
      body: data,
    });
    if (!res.ok) throw new Error();
    form.reset();
    formStatus.textContent = "Message envoyé, merci !";
    formStatus.classList.add("ok");
  } catch {
    formStatus.textContent = "Erreur d'envoi, réessaie dans un instant.";
    formStatus.classList.add("error");
  }
});

document.getElementById("year").textContent = new Date().getFullYear();
