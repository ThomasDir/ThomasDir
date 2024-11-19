<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestion Assainissement</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/tailwindcss/2.2.19/tailwind.min.css" rel="stylesheet">
    <link rel="manifest" href="data:application/json;base64,ewogICJuYW1lIjogIkdlc3Rpb24gQXNzYWluaXNzZW1lbnQiLAogICJzaG9ydF9uYW1lIjogIkFzc2Fpbmlzc2VtZW50IiwKICAic3RhcnRfdXJsIjogIi4vaW5kZXguaHRtbCIsCiAgImRpc3BsYXkiOiAic3RhbmRhbG9uZSIsCiAgImJhY2tncm91bmRfY29sb3IiOiAiI2ZmZmZmZiIsCiAgInRoZW1lX2NvbG9yIjogIiMwMDAwMDAiCn0=">
</head>
<body class="bg-gray-100">
    <div class="container mx-auto px-4 py-8">
        <h1 class="text-2xl font-bold mb-6">Gestion Assainissement</h1>
        
        <div class="bg-white rounded-lg shadow p-6 mb-6">
            <form id="equipementForm" class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700">Type d'équipement</label>
                    <select id="type" required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500">
                        <option value="">Sélectionner...</option>
                        <option value="canalisation">Canalisation</option>
                        <option value="grille">Grille</option>
                        <option value="bouche">Bouche d'égout</option>
                        <option value="regard">Regard</option>
                    </select>
                </div>

                <div>
                    <label class="block text-sm font-medium text-gray-700">État</label>
                    <select id="etat" required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500">
                        <option value="">Sélectionner...</option>
                        <option value="bon">Bon état</option>
                        <option value="surveillance">À surveiller</option>
                        <option value="reparation">Nécessite réparation</option>
                        <option value="urgent">Urgent</option>
                    </select>
                </div>

                <div>
                    <label class="block text-sm font-medium text-gray-700">Commentaire</label>
                    <textarea id="commentaire" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500"></textarea>
                </div>

                <div>
                    <label class="block text-sm font-medium text-gray-700">Photo</label>
                    <input type="file" id="photo" accept="image/*" capture="environment" class="mt-1 block w-full">
                    <img id="preview" class="mt-2 hidden max-w-full h-48 object-contain">
                </div>

                <div>
                    <button type="button" id="location" class="bg-blue-500 text-white px-4 py-2 rounded-md hover:bg-blue-600">
                        Obtenir la position GPS
                    </button>
                    <div id="coordinates" class="mt-2 text-sm text-gray-600"></div>
                </div>

                <button type="submit" class="w-full bg-green-500 text-white px-4 py-2 rounded-md hover:bg-green-600">
                    Enregistrer
                </button>
            </form>
        </div>

        <div id="liste" class="space-y-4">
            <!-- Les équipements seront affichés ici -->
        </div>
    </div>

    <script>
        let latitude = null;
        let longitude = null;

        // Gestion de la photo
        document.getElementById('photo').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    const preview = document.getElementById('preview');
                    preview.src = e.target.result;
                    preview.classList.remove('hidden');
                }
                reader.readAsDataURL(file);
            }
        });

        // Gestion de la géolocalisation
        document.getElementById('location').addEventListener('click', function() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(function(position) {
                    latitude = position.coords.latitude;
                    longitude = position.coords.longitude;
                    document.getElementById('coordinates').textContent = 
                        `Latitude: ${latitude}, Longitude: ${longitude}`;
                }, function(error) {
                    alert('Erreur de géolocalisation: ' + error.message);
                });
            } else {
                alert('Géolocalisation non supportée par votre navigateur');
            }
        });

        // Gestion du formulaire
        document.getElementById('equipementForm').addEventListener('submit', function(e) {
            e.preventDefault();

            if (!latitude || !longitude) {
                alert('Veuillez obtenir la position GPS');
                return;
            }

            const equipement = {
                type: document.getElementById('type').value,
                etat: document.getElementById('etat').value,
                commentaire: document.getElementById('commentaire').value,
                photo: document.getElementById('preview').src,
                latitude: latitude,
                longitude: longitude,
                date: new Date().toISOString()
            };

            // Sauvegarder dans le stockage local
            const equipements = JSON.parse(localStorage.getItem('equipements') || '[]');
            equipements.push(equipement);
            localStorage.setItem('equipements', JSON.stringify(equipements));

            // Réinitialiser le formulaire
            e.target.reset();
            document.getElementById('preview').classList.add('hidden');
            document.getElementById('coordinates').textContent = '';
            latitude = null;
            longitude = null;

            // Actualiser la liste
            afficherEquipements();
            alert('Équipement enregistré avec succès !');
        });

        // Afficher les équipements
        function afficherEquipements() {
            const equipements = JSON.parse(localStorage.getItem('equipements') || '[]');
            const liste = document.getElementById('liste');
            liste.innerHTML = '';

            equipements.reverse().forEach(function(equip) {
                const div = document.createElement('div');
                div.className = 'bg-white rounded-lg shadow p-4';
                div.innerHTML = `
                    <div class="flex justify-between items-start">
                        <div>
                            <h3 class="font-bold">${equip.type}</h3>
                            <p class="text-sm text-gray-600">État: ${equip.etat}</p>
                            <p class="text-sm text-gray-600">Date: ${new Date(equip.date).toLocaleString()}</p>
                            <p class="text-sm text-gray-600">Position: ${equip.latitude}, ${equip.longitude}</p>
                            <p class="text-sm mt-2">${equip.commentaire}</p>
                        </div>
                        <img src="${equip.photo}" class="w-24 h-24 object-cover rounded">
                    </div>
                `;
                liste.appendChild(div);
            });
        }

        // Initialiser l'affichage
        afficherEquipements();

        // Service Worker pour le mode hors-ligne
        if ('serviceWorker' in navigator) {
            navigator.serviceWorker.register('data:application/javascript;base64,c2VsZi5hZGRFdmVudExpc3RlbmVyKCdpbnN0YWxsJywgZnVuY3Rpb24oZXZlbnQpIHsKICBldmVudC53YWl0VW50aWwoCiAgICBjYWNoZXMub3BlbigndjEnKS50aGVuKGZ1bmN0aW9uKGNhY2hlKSB7CiAgICAgIHJldHVybiBjYWNoZS5hZGRBbGwoWycuLyddKTsKICAgIH0pCiAgKTsKfSk7CgpzZWxmLmFkZEV2ZW50TGlzdGVuZXIoJ2ZldGNoJywgZnVuY3Rpb24oZXZlbnQpIHsKICBldmVudC5yZXNwb25kV2l0aCgKICAgIGNhY2hlcy5tYXRjaChldmVudC5yZXF1ZXN0KS50aGVuKGZ1bmN0aW9uKHJlc3BvbnNlKSB7CiAgICAgIHJldHVybiByZXNwb25zZSB8fCBmZXRjaChldmVudC5yZXF1ZXN0KTsKICAgIH0pCiAgKTsKfSk7');
        }
    </script>
</body>
</html>
