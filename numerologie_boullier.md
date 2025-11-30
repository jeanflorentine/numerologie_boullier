import random
import re
from num2words import num2words
from colorama import init, Fore, Style

# Initialisation pour Windows
init(convert=True)

# Palette cyclique à haut contraste pour une différenciation parfaite
palette_cycle = [
    Fore.YELLOW,
    Fore.CYAN,
    Fore.LIGHTMAGENTA_EX
]

# Plages pour varier la taille des nombres
plages = [
    (1, 9),
    (10, 999),
    (1_000, 999_999),
    (1_000_000, 999_999_999),
    (1_000_000_000, 10_000_000_000)
]

def generate_random_number(plages):
    """Choisit une plage aléatoire et génère un nombre."""
    min_val, max_val = random.choice(plages)
    return random.randint(min_val, max_val)

def format_colored_number(nombre, palette):
    """Formate le nombre avec une palette de couleurs cyclique."""
    nombre_str = f"{nombre:,}".replace(",", ".")
    colored_output = []
    pos = 0

    for char in nombre_str[::-1]:
        if char.isdigit():
            couleur = palette[pos % len(palette)]
            colored_output.append(f"{couleur}{char}{Style.RESET_ALL}")
            pos += 1
        elif char == ".":
            # Le point prend la couleur du chiffre précédent (à droite)
            couleur = palette[(pos - 1) % len(palette)]
            colored_output.append(f"{couleur}.{Style.RESET_ALL}")
    
    return "".join(reversed(colored_output))

def creer_mots_colores(nombre, palette, lang):
    """Crée une chaîne de mots colorés synchronisée avec une palette cyclique."""
    if nombre == 0:
        return num2words(0, lang=lang)

    mots_complets = []
    milliards = nombre // 1_000_000_000
    millions = (nombre % 1_000_000_000) // 1_000_000
    mille = (nombre % 1_000_000) // 1_000
    reste = nombre % 1_000

    def ajouter_segment(valeur, nom_magnitude, puissance_base):
        if valeur > 0:
            mots_segment = num2words(valeur, lang=lang)
            if lang == 'fr' and valeur == 1 and nom_magnitude == 'mille':
                mots_segment = ''
            
            power = len(str(valeur)) - 1 + puissance_base
            couleur = palette[power % len(palette)]
            mots_complets.append(f"{couleur}{mots_segment}{Style.RESET_ALL}")
            
            if nom_magnitude:
                if lang == 'fr' and valeur > 1 and nom_magnitude in ['million', 'milliard']:
                    mots_complets.append(f"{nom_magnitude}s")
                else:
                    mots_complets.append(nom_magnitude)

    if lang == 'fr':
        ajouter_segment(milliards, 'milliard', 9)
        ajouter_segment(millions, 'million', 6)
        ajouter_segment(mille, 'mille', 3)
    elif lang == 'en':
        ajouter_segment(milliards, 'billion', 9)
        ajouter_segment(millions, 'million', 6)
        ajouter_segment(mille, 'thousand', 3)
    
    if reste > 0:
        mots_segment = num2words(reste, lang=lang)
        power = len(str(reste)) - 1
        couleur = palette[power % len(palette)]
        mots_complets.append(f"{couleur}{mots_segment}{Style.RESET_ALL}")
    
    return " ".join(mots_complets)


def afficher_legende(palette):
    """Affiche la légende complète des couleurs en utilisant la palette cyclique."""
    print("----------- Légende des Couleurs -----------")
    legende = [
        "UNITÉ (10^0)",
        "DIZAINE (10^1)",
        "CENTAINE (10^2)",
        "MILLIER (10^3)",
        "DIZAINE DE MILLE (10^4)",
        "CENTAINE DE MILLE (10^5)",
        "MILLION (10^6)",
        "DIZAINE DE MILLION (10^7)",
        "CENTAINE DE MILLION (10^8)",
        "MILLIARD (10^9)"
    ]
    for i, nom in enumerate(legende):
        couleur = palette[i % len(palette)]
        print(f"{couleur}{nom}{Style.RESET_ALL}")
    print("-------------------------------------------")


def main():
    """Fonction principale du programme."""
    afficher_legende(palette_cycle)
    nombre_de_tirages = 15
    for i in range(nombre_de_tirages):
        print(f"\n--- Tirage {i + 1}/{nombre_de_tirages} ---")
        nombre = generate_random_number(plages)
        
        colored_nombre_str = format_colored_number(nombre, palette_cycle)
        print(f"CHIFFRES:\t{colored_nombre_str}")

        mots_fr = creer_mots_colores(nombre, palette_cycle, 'fr')
        print(f"FRANCAIS:\t{mots_fr}")
        
        mots_en = creer_mots_colores(nombre, palette_cycle, 'en')
        print(f"ANGLAIS:\t{mots_en}")

        if i < nombre_de_tirages - 1:
            entree = input("Appuyez sur ENTER pour continuer ou 'q' pour quitter : ")
            if entree.lower() == 'q':
                print("Programme interrompu par l'utilisateur.")
                break
    
    print("\n15 tirages effectués. Programme terminé.")

if __name__ == "__main__":
    main()


