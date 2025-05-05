# Jeu-des-fourmis
import tkinter as tk
import json

root = tk.Tk()
root.title("Jeu des fourmis")

en_lecture = True  
id_animation = None  #stockera l'identifiant de l'animation
historique = [] 


CANVAS_WIDTH, CANVA_HEIGHT = 600, 400
canvas = tk.Canvas(root, width=CANVAS_WIDTH, height=CANVA_HEIGHT,bg="black",bd=2)
canvas.grid(row=0, column=0) 


taille_case = 40 
nb_colonnes = CANVAS_WIDTH // taille_case
nb_lignes = CANVA_HEIGHT // taille_case


# 0 = blanc, 1 = noir 
grille = [[0 for _ in range(nb_colonnes)] for _ in range(nb_lignes)]
#le "[0 for _ in range(nb_colonnes)]" cree une ligne remplie de 0 (blanc)
#le "for _ in range(nb_lignes)" le repete pour chaque lignes
 

def dessiner_grille():
    for y in range(nb_lignes):
        for x in range(nb_colonnes): 
            couleur = "black" if grille[y][x] == 1 else "white"
            canvas.create_rectangle(
                x * taille_case, y * taille_case, #coordonnées du coin supérieur gauche
                (x + 1) * taille_case, (y + 1) * taille_case, #coordonnées du coin inférieur droit
                fill=couleur, outline="gray", width=2
            )


  

position_fourmis = [nb_colonnes // 2, nb_lignes // 2]
direction = "haut"

def fourmis(): 
    x, y = position_fourmis
    taille_fourmis = taille_case // 3  
    x0 = position_fourmis[0] * taille_case + (taille_case - taille_fourmis) // 2
    y0 = position_fourmis[1] * taille_case + (taille_case - taille_fourmis) // 2
    x1 = x0 + taille_fourmis
    y1 = y0 + taille_fourmis
    canvas.create_oval(x0,y0, x1, y1, fill="red")


rotations = {
    "haut": {"gauche": "gauche", "droite": "droite"},
    "gauche": {"gauche": "bas", "droite": "haut"},
    "droite": {"gauche": "haut", "droite": "bas"},
    "bas": {"gauche": "droite", "droite": "gauche"},
}


def etape_suivante():
    global direction, position_fourmis, grille, historique   
    #on sauvegarde l'etat actuel avant de bouger
    historique.append({  
        "grille": [ligne.copy() for ligne in grille], 
        "position": position_fourmis.copy(),            
        "direction": direction                         
    }) #on doit faire des copies sinon python va modifier les originaux en meme temps
    
    x, y = position_fourmis
    #on lit la couleur actuelle 
    couleur_case = grille[y][x]  

    #on change la couleur de la case
    grille[y][x] = 1 - grille[y][x] 

    #on tourne selon l'ancienne couleur 
    if couleur_case == 1:  
        direction = rotations[direction]["gauche"]  
    else:  
        direction = rotations[direction]["droite"] 
    
    #on fais avancer la fourmis
    if direction == "haut":
        y -= 1  #monte d'une ligne
    elif direction == "bas":
        y += 1  #descends d'une ligne
    elif direction == "gauche":
        x -= 1  #va a gauche
    elif direction == "droite":
        x += 1  #va a droite
    
    #on gere le tore
    if x < 0:
        x = nb_colonnes - 1
    elif x >= nb_colonnes:
        x = 0

    if y < 0:
        y = nb_lignes - 1
    elif y >= nb_lignes:
        y = 0
    position_fourmis = [x, y]  #on met a jour la position
    
    #mise a jour de l'affichage 
    canvas.delete("all")      
    dessiner_grille()       
    fourmis()              
    #on nettoit le canva puis on redessine tout avec les nouvelles position/couleur.

def animer():
    global id_animation
    
    if en_lecture:  # si on est en mode lecture
        etape_suivante()  #on fait un pas
        id_animation = root.after(vitesse.get(), animer)
        #on utilise id comme ca on peux stopper l'animation (foncton pause)

def play():
    global en_lecture, id_animation
    
    if not en_lecture:  #si ce n'etait pas deja en lecture
        en_lecture = True
        animer()  #lance l'animation

def pause():
    global en_lecture, id_animation
    
    if en_lecture:  #qsi c'était en lecture
        en_lecture = False
        if id_animation:  
            root.after_cancel(id_animation) #annule l'animations en cours

def etape_precedente():
    global grille, position_fourmis, direction, historique
    
    if historique:  #si l'historique n'est pas vide
        etat_precedent = historique.pop()  #on recupere le dernier etat
        #on restaure tout
        grille = [ligne.copy() for ligne in etat_precedent["grille"]] 
        position_fourmis = etat_precedent["position"].copy()
        direction = etat_precedent["direction"]
        #on remet les copies qu'on a fais dans etape_suivante, ici 


        #on maj l'affichage
        canvas.delete("all")
        dessiner_grille()
        fourmis()

def sauvegarder():
    # Sauvegarde l'état actuel dans un fichier JSON
    etat = {
        "position": position_fourmis,
        "direction": direction,
        "grille": grille,
        "historique": historique
    }
    
    with open("sauvegarde_fourmis.json", "w") as f:
        json.dump(etat, f)  # Écrit en format JSON
    
    print("✅ Partie sauvegardée dans 'sauvegarde_fourmis.json'")

def charger():
    # Charge une partie depuis le fichier JSON
    global position_fourmis, direction, grille, historique, en_lecture, id_animation
    
    try:
        #stop l'animation si elle est en cours
        if id_animation:
            root.after_cancel(id_animation)
            en_lecture = False
        
        #charge le fichier
        with open("sauvegarde_fourmis.json", "r") as f:
            etat = json.load(f)
        
        # met à jour toute les variables
        position_fourmis = etat["position"]
        direction = etat["direction"]
        grille = etat["grille"]
        historique = etat.get("historique", [])
        
        #réaffiche tout
        canvas.delete("all")
        dessiner_grille()
        fourmis()  
        
        print("✅ Partie chargée avec succès!")
        
    except FileNotFoundError:
        print("Aucune sauvegarde trouvée !")
    except Exception as e:
        print(f"Erreur : {str(e)}")
    print("Position après chargement:", position_fourmis)
    print("Direction après chargement:", direction)
    print("Première ligne de la grille:", grille[0][:5])
        
#initialisation
grille[position_fourmis[1]][position_fourmis[0]] = 1  
dessiner_grille()
fourmis()



btn_load = tk.Button(root,text="Charger", bg="lightgray", font=("Arial", 12, "bold") )
btn_save = tk.Button(root,text= "Sauvegarder" , bg="lightgray", font=("Arial", 12, "bold"))
btn_back = tk.Button(root, text= "↶",bg="lightblue", font=("Arial", 12, "bold"))
btn_play = tk.Button(root, text="Play", bg="lightblue", font=("Arial", 12, "bold"))
btn_pause = tk.Button(root, text="Pause", bg="lightblue", font=("Arial", 12, "bold"))
btn_next = tk.Button(root, text="Next", bg="lightblue", font=("Arial", 12, "bold"))


 
btn_load.grid(row= 9,column=0,sticky="ew", padx=5, pady=5)
btn_save.grid(row=8 , column=0,sticky="ew", padx=5, pady=5 )
btn_back.grid(row=7, column = 0 , sticky="ew", padx=5, pady=5)
btn_play.grid(row=1, column=0, sticky="ew", padx=5, pady=5)
btn_pause.grid(row=2, column=0, sticky="ew", padx=5, pady=5)
btn_next.grid(row=3, column=0, sticky="ew", padx=5, pady=5)        

#slider pour la vitesse
vitesse = tk.IntVar(value=300)  #valeur par defaut 300 ms
slider = tk.Scale(
    root,
    from_=50,  #vitesse max 
    to=1000,   #vitesse min
    orient="horizontal",
    label="Vitesse (ms)",
    variable=vitesse,
    bg="lightgray"
)
slider.grid(row=4, column=0, sticky="ew", padx=5, pady=5)

#lier les boutons a leur fonctions coreesspondantes 
btn_back.config(command=etape_precedente)
btn_next.config(command=etape_suivante)
btn_play.config(command=play)
btn_pause.config(command=pause)
btn_load.config(command=charger)


root.mainloop() 


