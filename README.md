<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Gestion des Formulaires</title>
  <style>
    body { font-family: Arial, sans-serif; line-height: 1.6; margin: 20px; background: #f9f9f9; }
    h1 { text-align: center; color: #333; }
    .form-container { max-width: 800px; margin: 0 auto; padding: 20px; background: #fff; border: 1px solid #ddd; border-radius: 5px; box-shadow: 0 4px 6px rgba(0,0,0,.1); }
    #places { font-size: 18px; font-weight: bold; text-align: center; margin-bottom: 20px; }
    label { display: block; margin-top: 10px; font-weight: bold; }
    input, select { width: 100%; padding: 10px; margin-bottom: 10px; border: 1px solid #ddd; border-radius: 4px; box-sizing: border-box; }
    button { padding: 10px 20px; margin: 10px 10px 10px 0; background: #4caf50; color: #fff; font-size: 16px; border: none; border-radius: 5px; cursor: pointer; }
    button:hover { background: #45a049; }
    button:disabled { background: #ccc; cursor: not-allowed; }
    .form-section { margin-top: 20px; }
    .form-wrapper { border: 1px solid #ccc; padding: 15px; margin-top: 20px; border-radius: 5px; }
    .form-wrapper h2 { margin: 0 0 10px; color: #007bff; }
    .flex-row { display: flex; gap: 20px; }
    .flex-row > div { flex: 1; }
    .urgence-title, .autorisation-parentale { font-size: 20px; font-weight: bold; text-align: center; margin: 30px 0 10px; text-transform: uppercase; }
    .autorisation-block { display: flex; gap: 20px; justify-content: center; margin-bottom: 10px; }
    .checkbox-options { display: flex; gap: 20px; justify-content: center; margin: 10px 0; }
    .age-display { font-style: italic; color: #555; margin: -5px 0 10px; }
    .success-message { text-align: center; color: green; font-weight: bold; margin-top: 20px; }
    /* Section Fiche Sanitaire */
    .fiche-sanitaire { margin-top: 30px; padding: 15px; border: 2px solid orange; border-radius: 5px; background: #fcf8e3; }
    .fiche-sanitaire h2 { margin-top: 0; color: #f0ad4e; text-align: center; }
  </style>
</head>
<body>
  <h1>Gestion des Formulaires</h1>
  <div class="form-container">
    <p id="places">Il reste encore 48 places.</p>
    <button id="add-form-button">Ajouter un formulaire</button>
    <button id="remove-form-button" disabled>Supprimer un formulaire</button>
    <div id="form-section" class="form-section"></div>
    <p id="success-message" class="success-message" style="display:none;">Formulaire enregistré avec succès !</p>
  </div>

  <script>
    document.addEventListener('DOMContentLoaded', () => {
      const MAX_FORMS = 3;
      let formCount = 0;
      let totalPlaces = parseInt(localStorage.getItem('places') || '48', 10);
      const formSection = document.getElementById('form-section');
      const addBtn = document.getElementById('add-form-button');
      const remBtn = document.getElementById('remove-form-button');
      const placesEl = document.getElementById('places');
      const successMsg = document.getElementById('success-message');

      function updatePlaces() {
        if (totalPlaces <= 0) {
          placesEl.textContent = "Il n'y a plus de places disponibles.";
          addBtn.disabled = true;
        } else {
          addBtn.disabled = false;
          placesEl.textContent = totalPlaces <= 10
            ? `⚠️ Plus que ${totalPlaces} place${totalPlaces>1?'s':''} disponibles !`
            : `Il reste encore ${totalPlaces} place${totalPlaces>1?'s':''}.`;
        }
        localStorage.setItem('places', totalPlaces);
      }
      updatePlaces();

      function calcAge(dateStr) {
        const today = new Date();
        const dob = new Date(dateStr);
        let age = today.getFullYear() - dob.getFullYear();
        const m = today.getMonth() - dob.getMonth();
        if (m < 0 || (m === 0 && today.getDate() < dob.getDate())) age--;
        return age;
      }

      function createForm(idx) {
        const f = document.createElement('form'); f.className = 'form-wrapper';
        f.innerHTML = `
          <h2>Formulaire ${idx}</h2>
          <div class="flex-row">
            <div><label for="nom${idx}">Nom :</label><input id="nom${idx}" required></div>
            <div><label for="prenom${idx}">Prénom :</label><input id="prenom${idx}" required></div>
          </div>
          <label for="sexe${idx}">Sexe :</label>
          <select id="sexe${idx}" required>
            <option value="" disabled selected>Choisir...</option>
            <option value="feminin">Féminin</option>
            <option value="masculin">Masculin</option>
          </select>
          <label for="date${idx}">Date de naissance :</label>
          <input type="date" id="date${idx}" required>
          <div id="age${idx}" class="age-display"></div>
          <label for="adresse${idx}">Adresse :</label><input id="adresse${idx}" required>
          <label for="email${idx}">Adresse mail :</label><input type="email" id="email${idx}" required>
          <div class="flex-row">
            <div><label for="taille${idx}">Taille (cm) :</label><input type="number" id="taille${idx}" required></div>
            <div><label for="pointure${idx}">Pointure :</label><input type="number" id="pointure${idx}" required></div>
          </div>
          <label for="etab${idx}">Établissement scolaire :</label><input id="etab${idx}" required>
          <h3>Responsable légal 1</h3>
          <div class="flex-row">
            <div><label>Nom :</label><input required></div>
            <div><label>Prénom :</label><input required></div>
            <div><label>Tél :</label><input type="tel" required></div>
          </div>
          <h3>Responsable légal 2</h3>
          <div class="flex-row">
            <div><label>Nom :</label><input required></div>
            <div><label>Prénom :</label><input required></div>
            <div><label>Tél :</label><input type="tel" required></div>
          </div>
          <div class="urgence-title">Autre personne à prévenir en cas d'urgence</div>
          <label>Nom :</label><input required><label>Prénom :</label><input required><label>Tél :</label><input type="tel" required>
          <label>Nom :</label><input required><label>Prénom :</label><input required><label>Tél :</label><input type="tel" required>
          <div class="autorisation-parentale">Autorisation parentale</div>
          <div class="autorisation-block"><label>Je soussigné(e) :</label><input placeholder="Nom" required><input placeholder="Prénom" required></div>
          <div class="autorisation-block"><label>Responsable de l'enfant :</label><input placeholder="Nom" required><input placeholder="Prénom" required></div>
          <p style="text-align:center;">Autorise mon enfant à quitter seul le stage 100 % sport en fin de journée, à 17 h :</p>
          <div class="checkbox-options"><label><input type="radio" name="sortie${idx}" value="oui" required> Oui</label><label><input type="radio" name="sortie${idx}" value="non" required> Non</label></div>
          <div class="autorisation-parentale">Autre personne autorisée à venir chercher mon enfant</div>
          <div class="flex-row">
            <div><input placeholder="Nom" required></div>
            <div><input placeholder="Prénom" required></div>
            <div><input type="tel" placeholder="Tél" required></div>
          </div>
          <button type="button" class="submit-button">Enregistrer</button>
        `;
        const dateInput = f.querySelector(`#date${idx}`);
        const ageDiv = f.querySelector(`#age${idx}`);
        dateInput.addEventListener('change', () => {
          ageDiv.textContent = dateInput.value ? `Âge : ${calcAge(dateInput.value)} an${calcAge(dateInput.value)>1?'s':''}` : '';
        });
        f.querySelector('.submit-button').addEventListener('click', () => {
          if (!f.checkValidity()) { f.reportValidity(); return; }
          totalPlaces = Math.max(0, totalPlaces-1); updatePlaces();
          successMsg.style.display = 'block'; setTimeout(() => successMsg.style.display='none', 3000);
          const enfantNom = f.querySelector(`#nom${idx}`).value;
          const enfantPrenom = f.querySelector(`#prenom${idx}`).value;
          const fiche = document.createElement('div'); fiche.className = 'fiche-sanitaire';
          fiche.innerHTML = `
            <h2>Fiche Sanitaire - Formulaire ${idx}</h2>
            <div class="flex-row">
              <div><strong>Nom (enfant) :</strong> ${enfantNom}</div>
              <div><strong>Prénom (enfant) :</strong> ${enfantPrenom}</div>
            </div>
            <div style="font-weight:bold; margin-top:15px;">Renseignements Médicaux</div>
            <p>L'enfant suit-il un traitement médical ? 
              <label><input type="radio" name="traitement${idx}" value="oui" required> Oui</label>
              <label><input type="radio" name="traitement${idx}" value="non" required> Non</label>
            </p>
          `;
          formSection.appendChild(fiche);
          f.remove(); formCount--; remBtn.disabled = formCount===0; addBtn.disabled = formCount===MAX_FORMS;
        });
        return f;
      }

      addBtn.addEventListener('click', () => { if (formCount < MAX_FORMS) { formSection.appendChild(createForm(++formCount)); remBtn.disabled = false; addBtn.disabled = formCount===MAX_FORMS; } });

      remBtn.addEventListener('click', () => { if (formCount>0) { formSection.lastElementChild.remove(); formCount--; addBtn.disabled = formCount===MAX_FORMS; remBtn.disabled = formCount===0; } });
    });
  </script>
</body>
</html>
