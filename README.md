# lanters
rpg
import random
import time
import os

# ============================================================
#             EMOTIONAL SPECTRUM: WAR OF LIGHT
#                    LANTERN RPG v2.0
# ============================================================

# ============================================================
# UTILITIES
# ============================================================

def clear():
    print("\n" * 40)


def pause():
    input("\nPress ENTER to continue...")


def slow(text, speed=0.008):
    for char in text:
        print(char, end="", flush=True)
        time.sleep(speed)
    print()


def title(text):
    clear()
    print("=" * 70)
    print(text.center(70))
    print("=" * 70)


def line():
    print("-" * 70)


def ask(prompt, choices):
    while True:
        print()
        print(prompt)

        for number, choice in enumerate(choices, 1):
            print(f"{number}. {choice}")

        try:
            answer = int(input("> "))

            if 1 <= answer <= len(choices):
                return answer

        except ValueError:
            pass

        print("Please choose a valid option.")


# ============================================================
# LANTERN DATA
# ============================================================

RINGS = {

    "Red": {
        "emotion": "Rage",
        "corps": "Red Lantern Corps",
        "color": "🔴",
        "hp": 115,
        "energy": 110,
        "attack": 20,
        "defense": 5,
        "powers": [
            "Rage Plasma",
            "Blood-Red Energy"
        ]
    },

    "Orange": {
        "emotion": "Avarice",
        "corps": "Orange Lantern Corps",
        "color": "🟠",
        "hp": 110,
        "energy": 115,
        "attack": 22,
        "defense": 4,
        "powers": [
            "Orange Constructs",
            "Construct Theft"
        ]
    },

    "Yellow": {
        "emotion": "Fear",
        "corps": "Sinestro Corps",
        "color": "🟡",
        "hp": 115,
        "energy": 120,
        "attack": 19,
        "defense": 8,
        "powers": [
            "Fear Constructs",
            "Fear Inducement"
        ]
    },

    "Green": {
        "emotion": "Willpower",
        "corps": "Green Lantern Corps",
        "color": "🟢",
        "hp": 120,
        "energy": 125,
        "attack": 18,
        "defense": 10,
        "powers": [
            "Hard-Light Constructs",
            "Force Field"
        ]
    },

    "Blue": {
        "emotion": "Hope",
        "corps": "Blue Lantern Corps",
        "color": "🔵",
        "hp": 125,
        "energy": 140,
        "attack": 13,
        "defense": 12,
        "powers": [
            "Hope Amplification",
            "Hope Restoration"
        ]
    },

    "Indigo": {
        "emotion": "Compassion",
        "corps": "Indigo Tribe",
        "color": "🟣",
        "hp": 120,
        "energy": 130,
        "attack": 16,
        "defense": 10,
        "powers": [
            "Teleportation",
            "Spectrum Mimicry"
        ]
    },

    "Violet": {
        "emotion": "Love",
        "corps": "Star Sapphires",
        "color": "💗",
        "hp": 120,
        "energy": 130,
        "attack": 18,
        "defense": 10,
        "powers": [
            "Violet Constructs",
            "Violet Crystal"
        ]
    },

    "Black": {
        "emotion": "Death",
        "corps": "Black Lantern Corps",
        "color": "⚫",
        "hp": 135,
        "energy": 125,
        "attack": 24,
        "defense": 5,
        "powers": [
            "Life Drain",
            "Death Energy"
        ]
    },

    "White": {
        "emotion": "Life",
        "corps": "White Lantern",
        "color": "⚪",
        "hp": 140,
        "energy": 150,
        "attack": 20,
        "defense": 14,
        "powers": [
            "Ringbreaker",
            "Life Erasure"
        ]
    },

    # Omega Corp uses white as its own color. It is NOT the White Lantern Corps.
    "Omega": {
        "emotion": "Unity / Absolute Spectrum Control",
        "corps": "Omega Corp",
        "color": "⚪",
        "hp": 250,
        "energy": 10**18,
        "attack": 10000000,
        "defense": 1000000,
        "powers": [
            "Omega Constructs",
            "Omega Spectrum Burst"
        ]
    },

    "Ultraviolet": {
        "emotion": "Hidden Negative Emotion",
        "corps": "Ultraviolet Corps",
        "color": "🟪",
        "hp": 125,
        "energy": 135,
        "attack": 22,
        "defense": 7,
        "powers": [
            "Unseen Light",
            "Negative Emotion"
        ]
    }
}


# ============================================================
# EMOTIONAL STATS
# ============================================================

EMOTIONS = [
    "rage",
    "avarice",
    "fear",
    "willpower",
    "hope",
    "compassion",
    "love",
    "death",
    "life",
    "negative"
]


RING_FROM_EMOTION = {
    "rage": "Red",
    "avarice": "Orange",
    "fear": "Yellow",
    "willpower": "Green",
    "hope": "Blue",
    "compassion": "Indigo",
    "love": "Violet",
    "death": "Black",
    "life": "White",
    "negative": "Ultraviolet"
}


# ============================================================
# PLAYER
# ============================================================

class Player:

    def __init__(self, name):

        self.name = name

        self.ring = None
        self.original_ring = None
        self.corps = None
        self.stolen_rings = set()
        self.orange_lantern = False
        self.white_lantern = False
        self.visited_planets = set()
        self.story_route = None
        self.omega_corp = False
        self.omega_members = []
        # Rings collected from enemies. Your original ring is never lost.
        self.owned_rings = set()

        self.level = 1
        self.xp = 0

        self.max_hp = 100
        self.hp = 100

        self.max_energy = 100
        self.energy = 100

        self.attack = 10
        self.defense = 5

        self.credits = 100

        # SECRET CHARACTER: KYLE
        # Entering "Kyle" automatically makes the player an overpowered
        # Green Lantern with unlimited energy and 10,000,000 damage attacks.
        self.kyle_mode = name.strip().lower() == "kyle"
        if self.kyle_mode:
            self.ring = "Green"
            self.original_ring = "Green"
            self.corps = RINGS["Green"]["corps"]
            self.owned_rings.add("Green")
            self.max_hp = 1000000000
            self.hp = self.max_hp
            self.max_energy = 10**18
            self.energy = self.max_energy
            self.attack = 10_000_000
            self.defense = 1_000_000

        self.rage = 0
        self.fear = 0
        self.hope = 0

        self.story_flags = set()

        self.emotions = {
            emotion: 0
            for emotion in EMOTIONS
        }

        self.relationships = {
            "Hal Jordan": 0,
            "John Stewart": 0,
            "Guy Gardner": 0,
            "Sinestro": 0,
            "Carol Ferris": 0,
            "Atrocitus": 0,
            "Larfleeze": 0,
            "Saint Walker": 0,
            "Indigo-1": 0,
            "Black Hand": 0,
            "Guardians": 0
        }

    # --------------------------------------------------------

    def setup_ring(self, ring):

        if self.kyle_mode:
            self.ring = "Green"
            self.original_ring = "Green"
            self.corps = RINGS["Green"]["corps"]
            return

        self.ring = ring
        self.original_ring = ring
        self.corps = RINGS[ring]["corps"]
        self.owned_rings.add(ring)

        data = RINGS[ring]

        self.max_hp = data["hp"]
        self.hp = self.max_hp

        self.max_energy = data["energy"]
        self.energy = self.max_energy

        self.attack = data["attack"]
        self.defense = data["defense"]

    # --------------------------------------------------------

    def change_ring(self, ring):

        if self.kyle_mode:
            print("\n💚 Kyle's Green Lantern ring cannot be replaced.")
            return

        self.ring = ring
        self.corps = RINGS[ring]["corps"]
        self.owned_rings.add(ring)

        data = RINGS[ring]

        self.max_energy = data["energy"]

        if self.energy > self.max_energy:
            self.energy = self.max_energy

        self.attack = data["attack"]
        self.defense = data["defense"]

    # --------------------------------------------------------

    def add_stolen_ring(self, ring):
        self.stolen_rings.add(ring)
        if ring == "Orange":
            self.orange_lantern = True
        if not self.omega_corp:
            required = {"Red", "Orange", "Yellow", "Green", "Blue", "Indigo", "Violet", "Black", "Ultraviolet"}
            owned = set(self.stolen_rings) | {self.original_ring}
            if required.issubset(owned):
                self.white_lantern = True
                self.stolen_rings.add("White")

    # --------------------------------------------------------

    def heal(self, amount):

        self.hp = min(self.max_hp, self.hp + amount)

    # --------------------------------------------------------

    def recharge(self, amount):

        self.energy = min(
            self.max_energy,
            self.energy + amount
        )

    # --------------------------------------------------------

    def xp_gain(self, amount):

        self.xp += amount

        needed = self.level * 100

        while self.xp >= needed:

            self.xp -= needed
            self.level += 1

            self.max_hp += 20
            self.max_energy += 15

            self.attack += 3
            self.defense += 2

            self.hp = self.max_hp
            self.energy = self.max_energy

            print()
            print("⭐" * 25)
            print(f"LEVEL UP! You are now LEVEL {self.level}!")
            print("⭐" * 25)

            needed = self.level * 100

    # --------------------------------------------------------

    def relationship(self, person, amount):

        self.relationships[person] += amount

    # --------------------------------------------------------

    def stats(self):

        title("YOUR LANTERN")

        data = RINGS[self.ring]

        print(f"Name:       {self.name}")
        print(f"Ring:       {data['color']} {self.ring}")
        print(f"Emotion:    {data['emotion']}")
        print(f"Corps:      {self.corps}")
        print(f"Level:      {self.level}")
        print(f"XP:         {self.xp}")
        print(f"HP:         {self.hp}/{self.max_hp}")
        print(f"Energy:     {self.energy}/{self.max_energy}")
        print(f"Attack:     {self.attack}")
        print(f"Defense:    {self.defense}")
        print(f"Credits:    {self.credits}")
        print("Rings Collected:")
        owned = set(getattr(self, "stolen_rings", set())) | {self.original_ring}
        for ring_name in RINGS:
            if ring_name in owned:
                print(f"  💍 {ring_name}")
        if self.white_lantern:
            print("  ⚪ WHITE LANTERN")

        line()

        print("POWERS")

        for power in data["powers"]:
            print(f"⚡ {power}")

        line()
        print("COLLECTED RINGS")
        for owned in sorted(self.owned_rings):
            print(f"💍 {RINGS[owned]['color']} {owned} — {RINGS[owned]['corps']}")

        line()

        print("EMOTIONAL SPECTRUM")

        for emotion, value in self.emotions.items():
            print(f"{emotion.capitalize():20} {value}")

        pause()


# ============================================================
# PERSONALITY TEST
# ============================================================

QUESTIONS = [

    (
        "Someone hurts someone you love. What do you do?",
        [
            ("Attack immediately.", "rage"),
            ("Protect them no matter what.", "love"),
            ("Stay calm and make a plan.", "willpower"),
            ("Try to understand what happened.", "compassion"),
            ("Make the attacker afraid.", "fear")
        ]
    ),

    (
        "You discover unlimited wealth.",
        [
            ("Take it.", "avarice"),
            ("Give it away.", "compassion"),
            ("Use it to protect people.", "willpower"),
            ("Use it to give people hope.", "hope"),
            ("Use it to protect the people I love.", "love")
        ]
    ),

    (
        "You are completely terrified.",
        [
            ("Fight anyway.", "willpower"),
            ("Become the source of fear.", "fear"),
            ("Become angry.", "rage"),
            ("Accept the darkness.", "death"),
            ("Believe things can get better.", "hope")
        ]
    ),

    (
        "A city will be destroyed unless you sacrifice yourself.",
        [
            ("Do it.", "willpower"),
            ("Find another way.", "hope"),
            ("Sacrifice myself.", "love"),
            ("Save whoever I can.", "compassion"),
            ("Death doesn't scare me.", "death")
        ]
    ),

    (
        "Someone betrays you.",
        [
            ("Destroy them.", "rage"),
            ("Make them afraid.", "fear"),
            ("Forgive them.", "compassion"),
            ("Try to understand them.", "hope"),
            ("Never trust anyone again.", "negative")
        ]
    ),

    (
        "What matters most?",
        [
            ("Power.", "avarice"),
            ("Courage.", "willpower"),
            ("Hope.", "hope"),
            ("Love.", "love"),
            ("Compassion.", "compassion"),
            ("Rage.", "rage")
        ]
    ),

    (
        "Someone offers you incredible power.",
        [
            ("Take it.", "avarice"),
            ("Refuse if it hurts people.", "compassion"),
            ("Take it only if necessary.", "willpower"),
            ("Use it to protect someone.", "love"),
            ("Take it before someone else does.", "fear")
        ]
    ),

    (
        "Everything has gone wrong.",
        [
            ("I keep fighting.", "willpower"),
            ("Tomorrow can be better.", "hope"),
            ("I become angry.", "rage"),
            ("I protect whoever remains.", "compassion"),
            ("I embrace the darkness.", "negative")
        ]
    ),

    (
        "You find an enemy begging for mercy.",
        [
            ("Finish them.", "rage"),
            ("Spare them.", "compassion"),
            ("Make them fear me.", "fear"),
            ("Give them another chance.", "hope"),
            ("Their fate doesn't matter.", "death")
        ]
    ),

    (
        "What scares you most?",
        [
            ("Being powerless.", "fear"),
            ("Losing someone I love.", "love"),
            ("Failing people who depend on me.", "willpower"),
            ("Having no hope.", "hope"),
            ("Losing control of myself.", "negative")
        ]
    )
]


def personality_test(player):

    title("THE RING'S JUDGMENT")

    slow("The emotional spectrum is examining your soul...")
    time.sleep(1)

    for question, answers in QUESTIONS:

        clear()

        print("=" * 70)
        print(question)
        print("=" * 70)

        for number, answer in enumerate(answers, 1):
            print(f"{number}. {answer[0]}")

        while True:

            try:
                choice = int(input("\n> "))

                if 1 <= choice <= len(answers):
                    break

            except ValueError:
                pass

            print("Invalid choice.")

        emotion = answers[choice - 1][1]

        player.emotions[emotion] += 1

        # Emotional side effects
        if emotion == "rage":
            player.rage += 1

        if emotion == "fear":
            player.fear += 1

        if emotion == "hope":
            player.hope += 1

    highest = max(
        player.emotions,
        key=player.emotions.get
    )

    ring = RING_FROM_EMOTION[highest]

    player.setup_ring(ring)

    title("THE RING HAS CHOSEN")

    data = RINGS[ring]

    print(f"{data['color']} {ring.upper()} LIGHT")
    print()
    print(f"Emotion: {data['emotion']}")
    print(f"Corps: {data['corps']}")

    print()
    print("YOUR POWERS:")

    for power in data["powers"]:
        print(f"⚡ {power}")

    print()

    slow(
        f"{player.name}... your emotional signature has been detected."
    )

    pause()


# ============================================================
# RING THEFT SYSTEM
# ============================================================

def enemy_ring_from_name(enemy_name):
    """Guess which Corps ring an enemy carries."""
    name = enemy_name.lower()
    if "red lantern" in name or "atrocitus" in name:
        return "Red"
    if "orange" in name or "larfleeze" in name:
        return "Orange"
    if "yellow" in name or "sinestro" in name:
        return "Yellow"
    if "green lantern" in name or "hal jordan" in name:
        return "Green"
    if "blue" in name or "saint walker" in name:
        return "Blue"
    if "indigo" in name:
        return "Indigo"
    if "violet" in name or "star sapphire" in name or "carol ferris" in name:
        return "Violet"
    if "black" in name or "black hand" in name:
        return "Black"
    if "white" in name:
        return "White"
    if "ultraviolet" in name:
        return "Ultraviolet"
    if "parallax" in name:
        return "Yellow"
    return None


def check_white_lantern(player):
    """Become a White Lantern after collecting every other ring color."""
    required = set(RINGS.keys()) - {"White"}

    if required.issubset(player.owned_rings):
        already_white = "White" in player.owned_rings
        player.owned_rings.add("White")

        if not already_white:
            title("⚪ WHITE LANTERN ASCENSION")
            print("You have collected every color of the Emotional Spectrum!")
            print()
            print("🌈 RED • ORANGE • YELLOW • GREEN • BLUE")
            print("🟣 INDIGO • VIOLET • BLACK • ULTRAVIOLET")
            print()
            print("⚪ ALL LIGHTS BECOME ONE.")
            print("You have become a WHITE LANTERN!")
            print()
            print("⚪ RINGBREAKER — Destroy an opponent's ring.")
            print("✨ LIFE ERASURE — One-shot an opponent.")
            player.story_flags.add("white_lantern")
            pause()

        # White becomes the active ring, while every collected ring remains owned.
        player.ring = "White"
        player.corps = RINGS["White"]["corps"]
        data = RINGS["White"]
        player.max_energy = max(player.max_energy, data["energy"])
        player.energy = player.max_energy
        player.max_hp = max(player.max_hp, data["hp"])
        player.hp = player.max_hp
        player.attack = max(player.attack, data["attack"])
        player.defense = max(player.defense, data["defense"])
        return True

    return False


def steal_ring(player, enemy):
    """Attempt to take an enemy's ring while keeping the player's own ring."""
    enemy_ring = enemy.get("ring") or enemy_ring_from_name(enemy["name"])

    if not enemy_ring or enemy_ring not in RINGS:
        print("\n❌ This enemy does not have a Corps ring you can steal.")
        pause()
        return False

    if enemy_ring in player.owned_rings:
        print(f"\n💍 You already possess the {enemy_ring} ring.")
        pause()
        return False

    print(f"\n{RINGS[enemy_ring]['color']} You reach for the enemy's {enemy_ring} ring!")
    print(f"The ring contains: {', '.join(RINGS[enemy_ring]['powers'])}")

    if player.kyle_mode:
        success = True
    else:
        health_ratio = enemy["hp"] / enemy["max_hp"]
        chance = 35 if health_ratio > 0.60 else 55 if health_ratio > 0.25 else 75
        success = random.randint(1, 100) <= chance

    if success:
        player.owned_rings.add(enemy_ring)
        check_white_lantern(player)
        enemy["ring_stolen"] = True
        player.story_flags.add("stole_ring")
        player.story_flags.add(f"stole_{enemy_ring.lower()}_ring")
        print("\n⚡ SUCCESS! You rip the ring free!")
        print(f"💍 {RINGS[enemy_ring]['color']} {enemy_ring} ring acquired!")
        print(f"⚡ You can now use {RINGS[enemy_ring]['powers'][0]} and {RINGS[enemy_ring]['powers'][1]}.")
        print(f"💚 Your original {player.original_ring} ring is STILL yours!")
        pause()
        return True

    print("\n💥 The enemy pulls their hand away! The ring rejects your attempt.")
    pause()
    return False


def use_stolen_power(player, enemy):
    """Use a ring that was actually stolen during combat.
    The player's original ring is never shown as a stolen ring.
    """
    available = sorted(
        r for r in getattr(player, "stolen_rings", set())
        if r in RINGS and r != "White"
    )

    if not available:
        print("\n❌ You have not stolen any usable rings yet.")
        pause()
        return

    choices = [f"{RINGS[r]['color']} {r} ring — {RINGS[r]['corps']}" for r in available]
    choices.append("Cancel")
    choice = ask("Which stolen ring do you want to use?", choices)

    if choice == len(choices):
        return

    ring = available[choice - 1]
    print(f"\n💍 You activate the stolen {ring} ring!")
    use_ring_power(player, enemy, ring_override=ring)


# ============================================================
# COMBAT
# ============================================================

def combat(player, enemy_name, enemy_hp, enemy_attack, reward):

    enemy = {
        "name": enemy_name,
        "hp": enemy_hp,
        "max_hp": enemy_hp,
        "attack": enemy_attack,
        "ring": enemy_ring_from_name(enemy_name),
        "ring_stolen": False
    }

    # Fighting is NEVER forced anymore.
    # Every encounter starts as a conversation. The player can talk,
    # fight, attempt to persuade the enemy, or walk away.

    while enemy["hp"] > 0 and player.hp > 0:

        title(f"ENCOUNTER: {enemy['name']}")

        print(
            f"{RINGS[player.ring]['color']} "
            f"{player.name}: "
            f"{player.hp}/{player.max_hp} HP"
        )
        print(f"⚡ Energy: {player.energy}/{player.max_energy}")
        print()
        print(f"👹 {enemy['name']}: {max(0, enemy['hp'])}/{enemy['max_hp']} HP")
        print()

        choice = ask(
            "The enemy is watching you. What do you do?",
            [
                "Talk",
                "Fight",
                "Try to persuade them to leave",
                "Walk away"
            ]
        )

        # --------------------------------------------------------
        # TALK
        # --------------------------------------------------------
        if choice == 1:
            title(f"TALKING TO {enemy['name'].upper()}")

            dialogue_choices = [
                "Why are you fighting?",
                "We don't have to be enemies.",
                "Tell me what you know about the War of Light.",
                "I'm giving you one chance to stand down."
            ]

            dialogue = ask("What do you say?", dialogue_choices)

            if dialogue == 1:
                print(f'\nYou ask, "Why are you fighting, {enemy["name"]}?"')
                print(f'{enemy["name"]} hesitates.')
                print('"Because everyone keeps telling us the other Corps are the enemy."')
                player.story_flags.add("heard_enemy_story")

            elif dialogue == 2:
                print('\nYou lower your ring.')
                print('"We do not have to solve every problem with a fight."')
                print(f'{enemy["name"]} looks surprised.')
                player.relationships["Guardians"] += 0
                player.story_flags.add("offered_peace")

            elif dialogue == 3:
                print('\nYou ask what they know about the War of Light.')
                print(f'{enemy["name"]} tells you:')
                print('"Someone is pushing the Corps toward war on purpose."')
                print('"The attacks are connected. Find the source."')
                player.story_flags.add("war_clue")

            else:
                print('\nYou say, "Stand down. This is your last chance."')
                print(f'{enemy["name"]} studies your ring.')

            pause()

        # --------------------------------------------------------
        # PERSUADE
        # --------------------------------------------------------
        elif choice == 3:
            title("A CHANCE FOR PEACE")

            print(f'{enemy["name"]} is not attacking.')
            print("You explain that every pointless battle makes the spectrum weaker.")

            # Kyle has overwhelming presence, but still gets a story choice.
            if player.kyle_mode:
                print('\n💚 Your massive emerald aura fills the area.')
                print(f'{enemy["name"]}: "Okay! Okay! We are done here."')
                print("You win the encounter without fighting.")
                player.story_flags.add("peaceful_victory")
                pause()
                return True

            success = random.randint(1, 100) <= 60

            if success:
                print(f'\n{enemy["name"]} lowers their weapon.')
                print('"Maybe you are right. I will stand down."')
                player.story_flags.add("peaceful_victory")
                pause()
                return True
            else:
                print(f'\n{enemy["name"]} shakes their head.')
                print('"I cannot walk away yet."')
                print("The choice to fight is still yours.")
                pause()

        # --------------------------------------------------------
        # WALK AWAY
        # --------------------------------------------------------
        elif choice == 4:
            title("YOU WALK AWAY")

            print(f'You leave {enemy["name"]} behind.')
            print("No fight. No forced battle.")
            print("Sometimes the strongest choice is refusing to fight.")
            player.story_flags.add("walked_away")
            pause()
            return True

        # --------------------------------------------------------
        # FIGHT
        # --------------------------------------------------------
        elif choice == 2:
            while enemy["hp"] > 0 and player.hp > 0:

                title(f"BATTLE: {enemy['name']}")

                print(
                    f"{RINGS[player.ring]['color']} "
                    f"{player.name}: "
                    f"{player.hp}/{player.max_hp} HP"
                )
                print(f"⚡ Energy: {player.energy}/{player.max_energy}")
                print()
                print(
                    f"👹 {enemy['name']}: "
                    f"{max(0, enemy['hp'])}/{enemy['max_hp']} HP"
                )
                print()

                fight_choice = ask(
                    "Choose your action:",
                    [
                        "Basic Attack",
                        "Ring Power",
                        "Use Stolen Ring Power",
                        "Attempt to Steal Their Ring",
                        "Defend",
                        "Recharge",
                        "Stop Fighting"
                    ]
                )

                defending = False

                if fight_choice == 1:
                    if player.kyle_mode:
                        damage = 10_000_000
                    else:
                        damage = random.randint(
                            player.attack,
                            player.attack + 10
                        )

                    enemy["hp"] -= damage
                    print(f"\n💥 You deal {damage:,} damage!")

                elif fight_choice == 2:
                    use_ring_power(player, enemy)

                elif fight_choice == 3:
                    use_stolen_power(player, enemy)

                elif fight_choice == 4:
                    if steal_ring(player, enemy):
                        return True

                elif fight_choice == 5:
                    defending = True
                    print("\n🛡️ You create a defensive stance.")

                elif fight_choice == 6:
                    amount = random.randint(20, 35)

                    if player.kyle_mode:
                        print("\n⚡ Kyle does not need to recharge.")
                        print("Your energy remains unlimited.")
                    else:
                        player.recharge(amount)
                        print(f"\n⚡ You recharge {amount} energy.")

                elif fight_choice == 7:
                    print("\nYou stop attacking.")
                    print("The fight is over because YOU chose to end it.")
                    pause()
                    return True

                if enemy["hp"] > 0:
                    damage = random.randint(
                        max(1, enemy["attack"] - 5),
                        enemy["attack"] + 8
                    )

                    if defending:
                        damage //= 2

                    damage -= player.defense

                    if damage < 1:
                        damage = 1

                    player.hp -= damage

                    # Kyle is intentionally absurdly overpowered.
                    if player.kyle_mode:
                        player.hp = player.max_hp
                        player.energy = player.max_energy
                        print(
                            f"👹 {enemy['name']} tries to attack, "
                            f"but Kyle's Green Lantern aura blocks it."
                        )
                    else:
                        print(
                            f"👹 {enemy['name']} deals "
                            f"{damage} damage!"
                        )

                time.sleep(0.35)

            if player.hp <= 0:
                title("DEFEATED")
                print("Your ring flickers.")
                print()
                print("The emotional spectrum has fallen silent.")
                pause()
                return False

            title("VICTORY")
            print(f"💥 {enemy['name']} has been defeated!")
            print(f"⭐ +{reward} XP")
            print(f"💰 +{reward // 2} credits")

            player.xp_gain(reward)
            player.credits += reward // 2

            pause()
            return True

    return player.hp > 0


# ============================================================
# RING STEALING / WHITE LANTERN POWERS
# ============================================================

def steal_ring(player, enemy):
    ring = enemy.get("ring")
    if not ring:
        print("\n❌ This opponent has no ring left to steal.")
        pause(); return False
    if ring == "Orange" and enemy.get("name") != "Larfleeze":
        print("\n🟠 Larfleeze is the only Orange Lantern you can fight.")
        pause(); return False
    if ring in getattr(player, "stolen_rings", set()):
        print(f"\nYou already have the {ring} ring.")
        pause(); return False
    success = player.kyle_mode or random.random() < 0.60
    if not success:
        print(f"\n💍 {enemy['name']} protects their {ring} ring!")
        pause(); return False
    player.add_stolen_ring(ring)
    player.owned_rings.add(ring)
    enemy["ring"] = None
    enemy["ring_stolen"] = True
    print(f"\n💍 You stole the {ring} ring from {enemy['name']}!")
    if ring == "Orange":
        print("🟠 LARFLEEZE'S RING IS YOURS!")
        print("You are now the ONLY Orange Lantern.")
    if player.white_lantern:
        print("\n⚪ YOU HAVE COLLECTED EVERY COLOR!")
        print("⚪ YOU ARE NOW A WHITE LANTERN!")
    print("\n🛑 THE FIGHT IS OVER!")
    print(f"{enemy['name']} can no longer fight without their ring.")
    pause()
    return True


def use_white_lantern_power(player, enemy):
    if not player.white_lantern:
        print("\n❌ You are not a White Lantern yet.")
        pause(); return
    choice = ask("White Lantern Power:", [
        "RINGBREAKER — Destroy their ring",
        "LIFE ERASURE — One-shot them"
    ])
    if choice == 1:
        enemy["ring"] = None
        print(f"\n⚪ RINGBREAKER destroys {enemy['name']}'s ring!")
    else:
        enemy["hp"] = 0
        print(f"\n✨ LIFE ERASURE one-shots {enemy['name']}!")
    pause()


def enforce_orange_uniqueness(enemy):
    if enemy.get("ring") == "Orange" and enemy.get("name") != "Larfleeze":
        enemy["ring"] = "Green"
    return enemy


# ============================================================
# RING POWERS
# ============================================================

def use_ring_power(player, enemy, ring_override=None):

    ring = ring_override or player.ring

    title(f"{RINGS[ring]['color']} {ring} LIGHT")

    powers = RINGS[ring]["powers"]

    for number, power in enumerate(powers, 1):
        print(f"{number}. {power}")

    try:
        choice = int(input("\n> "))

    except ValueError:
        print("Invalid.")
        return

    if choice not in [1, 2]:
        print("Invalid.")
        return

    if not player.kyle_mode and player.energy < 20:
        print("\n❌ Not enough energy!")
        return

    # ========================================================
    # KYLE SECRET MODE — EVERY DAMAGE-DEALING ATTACK IS 10M
    # ========================================================
    # This applies even when Kyle is using a stolen ring.
    if player.kyle_mode:
        if ring == "White":
            if choice == 1:
                enemy_ring = enemy.get("ring") or enemy_ring_from_name(enemy["name"])
                if enemy_ring and enemy_ring in RINGS:
                    enemy["ring_stolen"] = True
                    enemy["ring"] = None
                    print(f"⚪ RINGBREAKER destroys {enemy["name"]}'s {enemy_ring} ring!")
                else:
                    print("⚪ RINGBREAKER finds no ring to destroy.")
            else:
                enemy["hp"] = 0
                print("✨ LIFE ERASURE hits the opponent for 10,000,000 damage!")
        else:
            enemy["hp"] -= 10_000_000
            print(f"💥 KYLE uses {ring} light for 10,000,000 damage!")
        print("⚡ Kyle's energy is unlimited.")
        pause()
        return

    # ========================================================
    # WHITE — ONLY TWO SPECIAL MOVES
    # ========================================================

    if ring == "White":
        if choice == 1:
            enemy_ring = enemy.get("ring") or enemy_ring_from_name(enemy["name"])
            if enemy_ring and enemy_ring in RINGS:
                enemy["ring_stolen"] = True
                enemy["ring"] = None
                player.owned_rings.add(enemy_ring)
                print(f"⚪ RINGBREAKER destroys {enemy['name']}'s {enemy_ring} ring!")
                print(f"💍 The destroyed ring is now part of your collection.")
            else:
                print("⚪ RINGBREAKER finds no Corps ring to destroy.")
            player.energy -= 30
        else:
            damage = 10_000_000
            enemy["hp"] -= damage
            player.energy -= 50
            print(f"✨ LIFE ERASURE deals {damage:,} damage — a one-shot attack!")
        pause()
        return

    # ========================================================
    # RED
    # ========================================================

    if ring == "Red":

        if choice == 1:

            damage = random.randint(35, 55)

            enemy["hp"] -= damage
            player.energy -= 20

            player.rage += 1

            print(
                f"🔥 RAGE PLASMA deals {damage} damage!"
            )

        else:

            damage = random.randint(45, 65)

            enemy["hp"] -= damage
            player.energy -= 30

            player.rage += 2

            print(
                f"🩸 BLOOD-RED ENERGY deals {damage} damage!"
            )

    # ========================================================
    # ORANGE
    # ========================================================

    elif ring == "Orange":

        if choice == 1:

            damage = random.randint(35, 60)

            enemy["hp"] -= damage
            player.energy -= 20

            print(
                f"🟠 ORANGE CONSTRUCT deals {damage} damage!"
            )

        else:

            damage = random.randint(40, 75)

            enemy["hp"] -= damage
            player.energy -= 35

            print(
                f"👻 CONSTRUCT THEFT drains {damage} power!"
            )

    # ========================================================
    # YELLOW
    # ========================================================

    elif ring == "Yellow":

        if choice == 1:

            damage = random.randint(35, 60)

            enemy["hp"] -= damage
            player.energy -= 20

            player.fear += 1

            print(
                f"🟡 FEAR CONSTRUCT deals {damage} damage!"
            )

        else:

            damage = random.randint(45, 70)

            enemy["hp"] -= damage
            player.energy -= 30

            player.fear += 2

            print(
                f"😨 FEAR INDUCEMENT terrifies the enemy!"
            )

            print(
                f"The enemy suffers {damage} damage!"
            )

    # ========================================================
    # GREEN
    # ========================================================

    elif ring == "Green":

        if choice == 1:

            damage = random.randint(35, 60)

            enemy["hp"] -= damage
            player.energy -= 20

            print(
                f"🟢 HARD-LIGHT CONSTRUCT deals {damage} damage!"
            )

        else:

            shield = random.randint(30, 50)

            player.hp = min(
                player.max_hp,
                player.hp + shield
            )

            player.energy -= 20

            print(
                f"🛡️ FORCE FIELD absorbs {shield} damage!"
            )

    # ========================================================
    # BLUE
    # ========================================================

    elif ring == "Blue":

        if choice == 1:

            heal = random.randint(35, 60)

            player.heal(heal)
            player.energy -= 20

            player.hope += 1

            print(
                f"💙 HOPE AMPLIFICATION restores {heal} HP!"
            )

        else:

            damage = random.randint(30, 55)

            enemy["hp"] -= damage
            player.energy -= 25

            print(
                f"🔵 HOPE ENERGY strikes for {damage} damage!"
            )

    # ========================================================
    # INDIGO
    # ========================================================

    elif ring == "Indigo":

        if choice == 1:

            damage = random.randint(35, 60)

            enemy["hp"] -= damage
            player.energy -= 25

            print(
                f"🌀 TELEPORTATION STRIKE deals {damage} damage!"
            )

        else:

            damage = random.randint(45, 75)

            enemy["hp"] -= damage
            player.energy -= 35

            print(
                f"🌈 SPECTRUM MIMICRY deals {damage} damage!"
            )

    # ========================================================
    # VIOLET
    # ========================================================

    elif ring == "Violet":

        if choice == 1:

            damage = random.randint(35, 60)

            enemy["hp"] -= damage
            player.energy -= 20

            print(
                f"💗 VIOLET CONSTRUCT deals {damage} damage!"
            )

        else:

            heal = random.randint(40, 65)

            player.heal(heal)
            player.energy -= 30

            print(
                f"💎 VIOLET CRYSTAL heals {heal} HP!"
            )

    # ========================================================
    # BLACK
    # ========================================================

    elif ring == "Black":

        if choice == 1:

            damage = random.randint(40, 70)

            enemy["hp"] -= damage

            player.hp = min(
                player.max_hp,
                player.hp + damage // 2
            )

            player.energy -= 25

            print(
                f"☠️ LIFE DRAIN deals {damage} damage "
                f"and restores your HP!"
            )

        else:

            damage = random.randint(50, 80)

            enemy["hp"] -= damage
            player.energy -= 40

            print(
                f"💀 DEATH ENERGY deals {damage} damage!"
            )

    # ========================================================
    # WHITE
    # ========================================================

    elif ring == "White":

        if choice == 1:

            heal = random.randint(50, 80)

            player.heal(heal)
            player.energy -= 25

            print(
                f"⚪ LIFE RESTORATION heals {heal} HP!"
            )

        else:

            damage = random.randint(45, 75)

            enemy["hp"] -= damage
            player.energy -= 30

            print(
                f"✨ LIFE ENERGY deals {damage} damage!"
            )

    # ========================================================
    # ULTRAVIOLET
    # ========================================================

    elif ring == "Ultraviolet":

        if choice == 1:

            damage = random.randint(40, 65)

            enemy["hp"] -= damage
            player.energy -= 25

            print(
                f"🟪 UNSEEN LIGHT deals {damage} damage!"
            )

        else:

            damage = random.randint(50, 85)

            enemy["hp"] -= damage
            player.energy -= 35

            player.emotions["negative"] += 1

            print(
                f"🌑 NEGATIVE EMOTION deals {damage} damage!"
            )

    pause()


# ============================================================
# CHARACTER ENCOUNTERS
# ============================================================

def hal_jordan(player):

    title("HAL JORDAN")

    slow(
        "A green streak cuts across the sky."
    )

    slow(
        "Hal Jordan lands in front of you."
    )

    print()

    print('"So... you are the new Lantern."')
    print('"Let me see what you can do."')

    choice = ask(
        "How do you respond?",
        [
            "Ask Hal for advice.",
            "Challenge Hal.",
            "Tell him you don't trust the Corps.",
            "Ask him about Parallax."
        ]
    )

    if choice == 1:

        player.relationship("Hal Jordan", 3)

        print("\nHal nods.")

        print(
            '"Good. A Lantern who knows when to learn '
            'is dangerous."'
        )

    elif choice == 2:

        player.relationship("Hal Jordan", 1)

        print("\nHal grins.")

        print('"I like your attitude."')

        combat(
            player,
            "Hal Jordan - Training",
            130,
            20,
            50
        )

    elif choice == 3:

        player.relationship("Hal Jordan", -2)

        print(
            '\nHal says, "You don\'t have to trust us."'
        )

        print(
            '"Just protect the innocent."'
        )

    else:

        player.relationship("Hal Jordan", 2)

        player.story_flags.add("asked_about_parallax")

        print()

        slow(
            '"Parallax is fear given form."'
        )

        slow(
            '"And if it ever gets inside your head..."'
        )

        slow(
            '"Run."'
        )

    pause()


def john_stewart(player):

    title("JOHN STEWART")

    slow(
        "John Stewart examines your ring carefully."
    )

    print()

    print('"A ring gives you power."')
    print('"Discipline decides what you do with it."')

    choice = ask(
        "What do you ask John?",
        [
            "Train me.",
            "Tell me about the Guardians.",
            "Teach me how to build better constructs."
        ]
    )

    if choice == 1:

        player.relationship("John Stewart", 3)

        player.defense += 3

        print("\n🛡️ Your defense increases!")

    elif choice == 2:

        player.relationship("Guardians", -1)

        print(
            '\nJohn says, "The Guardians have made mistakes."'
        )

    else:

        player.relationship("John Stewart", 3)

        player.attack += 3

        print(
            "\n🏗️ John teaches you precision construction."
        )

    pause()


def guy_gardner(player):

    title("GUY GARDNER")

    slow(
        "Guy Gardner looks you up and down."
    )

    print()

    print('"You look like you need a little attitude."')

    choice = ask(
        "What do you do?",
        [
            "Insult him.",
            "Ask him to train you.",
            "Laugh."
        ]
    )

    if choice == 1:

        player.relationship("Guy Gardner", 2)

        print(
            "\nGuy laughs loudly."
        )

    elif choice == 2:

        player.relationship("Guy Gardner", 3)

        player.attack += 4

        print(
            "\n💥 Guy teaches you aggressive Lantern combat!"
        )

    else:

        player.relationship("Guy Gardner", 2)

    pause()


def sinestro(player):

    title("SINESTRO")

    slow(
        "The room suddenly becomes silent."
    )

    slow(
        "Sinestro steps out of the shadows."
    )

    print()

    print('"Fear is not weakness."')
    print('"Fear is power."')

    choice = ask(
        "How do you respond?",
        [
            "Agree with him.",
            "Tell him fear should never control people.",
            "Ask him about Parallax.",
            "Ask why he created the Sinestro Corps."
        ]
    )

    if choice == 1:

        player.relationship("Sinestro", 3)

        player.fear += 1

    elif choice == 2:

        player.relationship("Sinestro", -3)

    elif choice == 3:

        player.relationship("Sinestro", 1)

        player.story_flags.add("knows_parallax")

        print()

        slow(
            '"Parallax is not merely a creature."'
        )

        slow(
            '"It is fear itself."'
        )

    else:

        player.relationship("Sinestro", 2)

        print()

        slow(
            '"The universe needed order."'
        )

        slow(
            '"The Guardians were too afraid to provide it."'
        )

    pause()


def carol_ferris(player):

    title("CAROL FERRIS")

    slow(
        "Violet energy fills the room."
    )

    slow(
        "Carol Ferris appears."
    )

    print()

    print('"Love is not weakness."')
    print('"People simply misunderstand it."')

    choice = ask(
        "What do you say?",
        [
            "Love is powerful.",
            "Love makes people vulnerable.",
            "I don't know what love means to me."
        ]
    )

    if choice == 1:

        player.relationship("Carol Ferris", 3)
        player.emotions["love"] += 1

    elif choice == 2:

        player.relationship("Carol Ferris", -1)

    else:

        player.relationship("Carol Ferris", 2)

    pause()


def atrocitus(player):

    title("ATROCITUS")

    slow(
        "The planet shakes."
    )

    slow(
        "A red figure emerges from the darkness."
    )

    print()

    print('"RAGE IS POWER."')

    choice = ask(
        "What do you do?",
        [
            "Embrace your rage.",
            "Tell Atrocitus rage controls you.",
            "Ask what created the Red Lanterns."
        ]
    )

    if choice == 1:

        player.relationship("Atrocitus", 3)
        player.rage += 2

    elif choice == 2:

        player.relationship("Atrocitus", -2)

    else:

        player.relationship("Atrocitus", 1)

    pause()


def larfleeze(player):

    title("LARFLEEZE")

    slow(
        '"MINE!"'
    )

    slow(
        '"EVERYTHING IS MINE!"'
    )

    choice = ask(
        "Larfleeze sees your ring. What do you do?",
        [
            "Give him a fake ring.",
            "Tell him no.",
            "Offer him credits."
        ]
    )

    if choice == 1:

        player.relationship("Larfleeze", 3)

        print(
            "\nLarfleeze runs away with the fake ring."
        )

    elif choice == 2:

        player.relationship("Larfleeze", -2)

        print(
            '\n"THEN I WILL TAKE IT!"'
        )

    else:

        player.credits = max(
            0,
            player.credits - 25
        )

        player.relationship("Larfleeze", 2)

        print(
            "\nLarfleeze accepts your payment."
        )

    pause()


def saint_walker(player):

    title("SAINT WALKER")

    slow(
        "Blue light surrounds you."
    )

    slow(
        "Saint Walker approaches."
    )

    print()

    print('"There is always hope."')

    choice = ask(
        "What do you say?",
        [
            "I believe you.",
            "Hope isn't enough.",
            "Teach me about hope."
        ]
    )

    if choice == 1:

        player.relationship("Saint Walker", 3)
        player.hope += 2

    elif choice == 2:

        player.relationship("Saint Walker", -1)

    else:

        player.relationship("Saint Walker", 4)
        player.hope += 2

    pause()


def indigo_one(player):

    title("INDIGO-1")

    slow(
        "A strange indigo light surrounds you."
    )

    slow(
        "Indigo-1 studies your emotions."
    )

    print()

    print('"You carry pain."')

    choice = ask(
        "How do you respond?",
        [
            "Everyone carries pain.",
            "I don't want to talk about it.",
            "Help me understand it."
        ]
    )

    if choice == 1:

        player.relationship("Indigo-1", 2)

    elif choice == 2:

        player.relationship("Indigo-1", -1)

    else:

        player.relationship("Indigo-1", 4)
        player.emotions["compassion"] += 2

    pause()


def black_hand(player):

    title("BLACK HAND")

    slow(
        "The lights begin to flicker."
    )

    slow(
        "A black figure appears."
    )

    print()

    print('"Everything alive will eventually die."')

    choice = ask(
        "What do you do?",
        [
            "Fight him.",
            "Ask him why he worships death.",
            "Tell him life is stronger."
        ]
    )

    if choice == 1:

        player.relationship("Black Hand", -3)

    elif choice == 2:

        player.relationship("Black Hand", 1)

    else:

        player.relationship("Black Hand", -2)
        player.hope += 1

    pause()


# ============================================================
# LOCATIONS
# ============================================================

def earth(player):

    title("🌎 EARTH")

    choice = ask(
        "Where do you go?",
        [
            "Coast City",
            "Ferris Air",
            "Metropolis",
            "Gotham",
            "Return to galaxy map"
        ]
    )

    if choice == 1:

        title("COAST CITY")

        slow(
            "You arrive in Hal Jordan's city."
        )

        hal_jordan(player)

    elif choice == 2:

        title("FERRIS AIR")

        carol_ferris(player)

    elif choice == 3:

        title("METROPOLIS")

        slow(
            "You patrol the city."
        )

        combat(
            player,
            "Fear Construct",
            130,
            21,
            75
        )

    elif choice == 4:

        title("GOTHAM")

        slow(
            "The city is unusually quiet."
        )

        combat(
            player,
            "Black Lantern",
            145,
            24,
            90
        )


def oa(player):

    title("🟢 OA")

    slow(
        "The emerald towers of Oa rise before you."
    )

    choice = ask(
        "Where do you go?",
        [
            "Central Power Battery",
            "Lantern Training Grounds",
            "Guardians' Chamber",
            "Meet Hal Jordan",
            "Meet John Stewart",
            "Meet Guy Gardner",
            "Return"
        ]
    )

    if choice == 1:

        title("CENTRAL POWER BATTERY")

        slow(
            "The massive battery illuminates Oa."
        )

        player.recharge(player.max_energy)

        print("\n⚡ Your ring is fully recharged.")

        pause()

    elif choice == 2:

        title("TRAINING GROUNDS")

        john_stewart(player)

    elif choice == 3:

        title("GUARDIANS' CHAMBER")

        slow(
            "The Guardians watch you carefully."
        )

        player.relationship("Guardians", 1)

        print()

        print('"Your emotional instability concerns us."')

        choice2 = ask(
            "How do you respond?",
            [
                "I will prove myself.",
                "You don't control me.",
                "I understand."
            ]
        )

        if choice2 == 1:
            player.relationship("Guardians", 2)

        elif choice2 == 2:
            player.relationship("Guardians", -3)

        else:
            player.relationship("Guardians", 1)

        pause()

    elif choice == 4:
        hal_jordan(player)

    elif choice == 5:
        john_stewart(player)

    elif choice == 6:
        guy_gardner(player)


def zamaron(player):

    title("💗 ZAMARON")

    slow(
        "Violet energy surrounds the planet."
    )

    carol_ferris(player)


def odym(player):

    title("🔵 ODYM")

    saint_walker(player)


def ysmault(player):

    title("🔴 YSMAULT")

    slow(
        "The world burns with red light."
    )

    atrocitus(player)

    combat(
        player,
        "Red Lantern Champion",
        160,
        26,
        100
    )


def orange_space(player):

    title("🟠 ORANGE SPACE")

    larfleeze(player)

    combat(
        player,
        "Orange Construct",
        140,
        23,
        100
    )


def indigo_space(player):

    title("🟣 INDIGO SPACE")

    indigo_one(player)


def black_zone(player):

    title("⚫ BLACK LANTERN ZONE")

    slow(
        "You enter a region where life itself feels wrong."
    )

    black_hand(player)

    combat(
        player,
        "Black Lantern Champion",
        175,
        28,
        125
    )


def qward(player):

    title("🟡 QWARD")

    slow(
        "Yellow energy illuminates the planet."
    )

    sinestro(player)

    combat(
        player,
        "Sinestro Corps Soldier",
        135,
        22,
        90
    )


# ============================================================
# GALAXY MAP
# ============================================================

def galaxy_map(player):

    while True:

        title("🌌 GALACTIC MAP")

        print("Choose your destination:")

        choices = [
            "🌎 Earth",
            "🟢 Oa",
            "💗 Zamaron",
            "🔵 Odym",
            "🔴 Ysmault",
            "🟠 Orange Space",
            "🟣 Indigo Space",
            "⚫ Black Lantern Zone",
            "🟡 Qward",
            "📊 Character Stats",
            "🚪 Leave Map"
        ]

        choice = ask("", choices)

        if choice == 1:
            earth(player)
            planet_to_planet_corps_encounters(player, "Earth")

        elif choice == 2:
            oa(player)
            planet_to_planet_corps_encounters(player, "Oa")

        elif choice == 3:
            zamaron(player)
            planet_to_planet_corps_encounters(player, "Zamaron")

        elif choice == 4:
            odym(player)
            planet_to_planet_corps_encounters(player, "Odym")

        elif choice == 5:
            ysmault(player)
            planet_to_planet_corps_encounters(player, "Ysmault")

        elif choice == 6:
            orange_space(player)
            planet_to_planet_corps_encounters(player, "Okaara")

        elif choice == 7:
            indigo_space(player)
            planet_to_planet_corps_encounters(player, "Indigo Space")

        elif choice == 8:
            black_zone(player)
            planet_to_planet_corps_encounters(player, "Black Lantern Zone")

        elif choice == 9:
            qward(player)
            planet_to_planet_corps_encounters(player, "Qward")

        elif choice == 10:
            player.stats()

        else:
            break


# ============================================================
# STORY CHAPTER 1
# ============================================================

def chapter_one(player):

    title("CHAPTER I — THE RING CHOOSES")

    slow(
        "The night sky suddenly changes."
    )

    slow(
        "Green."
    )

    slow(
        "Red."
    )

    slow(
        "Yellow."
    )

    slow(
        "Blue."
    )

    slow(
        "Every color appears at once."
    )

    print()

    slow(
        "Then everything goes black."
    )

    print()

    slow(
        "A voice whispers..."
    )

    slow(
        '"The emotional spectrum is breaking."'
    )

    pause()

    if player.kyle_mode:
        title("SECRET IDENTITY DETECTED")
        slow("Kyle...")
        slow("Your Green Lantern ring does not need a test.")
        slow("It already knows exactly who you are.")
        slow("Willpower: UNLIMITED.")
        slow("Energy: UNLIMITED.")
        slow("Attack Power: 10,000,000.")
        print()
        print("💚 You have been chosen as an exceptionally powerful Green Lantern.")
        pause()
    else:
        personality_test(player)


# ============================================================
# STORY CHAPTER 2
# ============================================================

def chapter_two(player):

    title("CHAPTER II — THE CALL OF OA")

    slow("Your ring suddenly activates.")
    slow('"ATTENTION, LANTERN."')
    slow('"REPORT TO OA IMMEDIATELY."')
    slow("The message is repeated across the Corps network.")

    print()
    slow("You travel toward Oa.")
    slow("As you approach, you see Lanterns from every Corps gathering.")
    slow("Nobody knows who started the conflict.")
    pause()

    title("THE OA COUNCIL")
    slow("The Guardians stand above the chamber.")
    slow("Lanterns from across the emotional spectrum begin shouting at each other.")

    print()
    print("🔴 RED: \"Green Lanterns caused this!\"")
    print("🟢 GREEN: \"Yellow Lanterns are spreading fear!\"")
    print("🟡 YELLOW: \"Green is hiding something!\"")
    print("🔵 BLUE: \"Everyone needs to calm down.\"")
    print("💗 VIOLET: \"You are letting emotion control you.\"")
    print("🟣 INDIGO: \"The anger is making the truth impossible to see.\"")
    print("⚫ BLACK: \"The dead are already watching.\"")
    print("🟠 LARFLEEZE: \"EVERYTHING IS MINE!\"")

    print()
    choice = ask(
        "Everyone turns toward you. What do you do?",
        [
            "Tell everyone to stop blaming each other.",
            "Ask each Corps what evidence they have.",
            "Accuse the Yellow Lanterns.",
            "Accuse the Green Lanterns.",
            "Stay quiet and listen."
        ]
    )

    if choice == 1:
        player.story_flags.add("oa_peacemaker")
        player.relationship("Guardians", 2)
        slow("You raise your ring.")
        slow('"Enough! We are being manipulated, and fighting each other is exactly what someone wants."')
    elif choice == 2:
        player.story_flags.add("oa_investigator")
        slow("You demand evidence instead of accusations.")
        slow("The chamber becomes quieter.")
        slow("Nobody can prove who started the attacks.")
    elif choice == 3:
        player.story_flags.add("blamed_yellow")
        player.relationship("Sinestro", -2)
        slow("The Yellow Lanterns immediately turn toward you.")
    elif choice == 4:
        player.story_flags.add("blamed_green")
        player.relationship("Hal Jordan", -2)
        slow("The Green Lanterns glare at you.")
    else:
        player.story_flags.add("listened_at_oa")
        slow("You say nothing.")
        slow("Instead, you listen to every accusation.")

    print()
    slow("Then the Guardians reveal the truth: someone is deliberately turning the Corps against one another.")
    slow("The War of Light has begun.")
    pause()

    oa(player)


# ============================================================
# STORY INTERLUDE — THE SILENCE BETWEEN LIGHTS
# ============================================================

def story_interlude(player):

    title("INTERLUDE — THE SILENCE BETWEEN LIGHTS")

    slow("You leave Oa, but the feeling that something is wrong follows you.")
    slow("For the first time, your ring does not give you an answer.")
    slow("It gives you a question.")

    print()
    print('"WHO DO YOU TRUST?"')
    print()

    choice = ask(
        "A strange signal reaches your ring. How do you respond?",
        [
            "Follow the signal.",
            "Contact Hal Jordan.",
            "Contact John Stewart.",
            "Ignore it and continue your mission."
        ]
    )

    if choice == 1:
        slow("\nYou follow the signal into a dead zone between sectors.")
        slow("A broken Lantern beacon floats in the darkness.")
        slow("Someone has deliberately destroyed it.")
        player.story_flags.add("found_beacon")

    elif choice == 2:
        player.relationship("Hal Jordan", 2)
        slow('\nHal answers immediately: "I felt it too."')
        slow('"Something is interfering with the Corps network."')
        player.story_flags.add("hal_contact")

    elif choice == 3:
        player.relationship("John Stewart", 2)
        slow('\nJohn says, "Do not assume every problem needs a weapon."')
        slow('"Investigate before you choose a side."')
        player.story_flags.add("john_contact")

    else:
        slow("\nYou ignore the signal.")
        slow("Sometimes a mystery becomes more dangerous when you refuse to look at it.")
        player.story_flags.add("ignored_signal")

    pause()


# ============================================================
# STORY CHAPTER 3

# ============================================================

def chapter_three(player):

    title("CHAPTER III — WAR OF LIGHT")

    slow("The Corps are blaming each other.")
    slow("Red blames Green.")
    slow("Yellow blames Green.")
    slow("Green blames Yellow.")
    slow("Black Lanterns are appearing everywhere.")

    print()
    slow("But you notice something strange...")
    slow("The enemies are not attacking you.")
    slow("They are waiting.")
    slow("Almost as if someone wants YOU to start the war.")

    pause()

    choice = ask(
        "How do you investigate?",
        [
            "Talk to the first Lantern patrol.",
            "Search for the source of the attacks.",
            "Prepare for a fight, but do not start one.",
            "Leave the sector and gather allies."
        ]
    )

    if choice == 1:
        combat(player, "Sinestro Corps Soldier", 130, 22, 75)

    elif choice == 2:
        title("THE HIDDEN SIGNAL")
        slow("You trace the attacks to a repeating energy signature.")
        slow("It is not Red.")
        slow("It is not Green.")
        slow("It is not Yellow.")
        slow("Something is manipulating the emotional spectrum.")
        player.story_flags.add("found_hidden_signal")
        pause()

    elif choice == 3:
        combat(player, "Red Lantern", 145, 24, 85)

    else:
        title("THE GATHERING")
        slow("You refuse to rush into a war.")
        slow("You travel between sectors, looking for people willing to listen.")
        player.story_flags.add("gathered_allies")
        player.relationship("Hal Jordan", 1)
        player.relationship("John Stewart", 1)
        pause()

    return player.hp > 0


# ============================================================
# STORY CHAPTER 4
# ============================================================

def chapter_four(player):

    title("CHAPTER IV — THE OTHER LIGHTS")

    slow(
        "The Guardians send you across the spectrum."
    )

    slow(
        "You must discover who is causing the instability."
    )

    pause()

    galaxy_map(player)

    return True


# ============================================================
# STORY CHAPTER 5
# ============================================================

def chapter_five(player):

    title("CHAPTER V — BLACK HAND")

    slow(
        "The dead are returning."
    )

    slow(
        "Black Lantern rings are multiplying."
    )

    slow(
        "Something much worse is coming."
    )

    pause()

    choice = ask(
        "Black Hand is waiting. What do you do?",
        [
            "Talk to Black Hand.",
            "Enter the Black Lantern Zone and investigate.",
            "Attack.",
            "Turn around and leave."
        ]
    )

    if choice == 1:
        black_hand(player)
    elif choice == 2:
        black_zone(player)
    elif choice == 3:
        combat(player, "Black Lantern Champion", 175, 28, 125)
    else:
        title("YOU LEAVE THE DARKNESS")
        slow("You refuse to enter the trap.")
        slow("But you know Black Hand will not simply disappear.")
        player.story_flags.add("avoided_black_zone")
        pause()

    return player.hp > 0


# ============================================================
# RING EVOLUTION
# ============================================================

def emotional_evolution(player):

    title("EMOTIONAL EVOLUTION")

    highest = max(
        player.emotions,
        key=player.emotions.get
    )

    possible_ring = RING_FROM_EMOTION[highest]

    print(
        f"Your strongest emotion is: "
        f"{highest.upper()}"
    )

    print(
        f"This emotion resonates with the "
        f"{possible_ring} light."
    )

    if possible_ring != player.ring:

        print()

        choice = ask(
            "Your emotional state is changing.",
            [
                f"Accept the {possible_ring} light.",
                "Remain loyal to my current ring."
            ]
        )

        if choice == 1:

            old = player.ring

            player.change_ring(possible_ring)

            print()
            print(
                f"💍 Your ring changes from "
                f"{old} to {possible_ring}!"
            )

            player.story_flags.add("changed_ring")

    else:

        print(
            "\nYour ring remains perfectly synchronized."
        )

    pause()


# ============================================================
# FINAL BATTLE — NORMAL
# ============================================================

def final_parallax(player):

    title("FINAL CHAPTER — PARALLAX")

    slow(
        "The universe becomes silent."
    )

    slow(
        "Every Lantern ring begins flashing."
    )

    slow(
        "Then a massive yellow figure appears."
    )

    print()

    print("                 👁️ PARALLAX 👁️")

    print()

    slow(
        '"I AM FEAR."'
    )

    slow(
        '"EVERY FEAR YOU HAVE EVER HAD."'
    )

    slow(
        '"EVERY FEAR YOU EVER WILL HAVE."'
    )

    pause()

    choice = ask(
        "Parallax waits before you. What do you do?",
        [
            "Talk to Parallax.",
            "Fight Parallax.",
            "Try to reach the fear inside it.",
            "Leave and regroup."
        ]
    )

    if choice == 1:
        title("THE VOICE OF FEAR")
        slow('Parallax laughs. "You think you can reason with fear?"')
        slow('You answer, "Fear can be understood."')
        slow("For one moment, the yellow monster becomes silent.")
        player.story_flags.add("spoke_to_parallax")
        pause()
        return True

    elif choice == 3:
        title("CONFRONTING FEAR")
        slow("You do not attack.")
        slow("You stand inside the storm and refuse to run.")
        slow("Parallax recoils from your willpower.")
        player.story_flags.add("confronted_fear")
        pause()
        return True

    elif choice == 4:
        title("REGROUP")
        slow("You retreat before the final confrontation.")
        slow("The war is not over. You simply chose when to fight.")
        player.story_flags.add("regrouped_before_final")
        pause()
        return True

    return combat(
        player,
        "PARALLAX",
        700,
        45,
        2000
    )


# ============================================================
# SPECIAL YELLOW ENDING
# ============================================================

def final_parallax_hal(player):

    title("SPECIAL FINAL CHAPTER")

    print()
    print("          🟡 PARALLAX — HAL JORDAN 🟡")
    print()

    slow(
        "You arrive in Coast City."
    )

    slow(
        "The entire city is frozen."
    )

    slow(
        "Every citizen is trapped inside their own fear."
    )

    print()

    slow(
        "Then you look up."
    )

    slow(
        "A green Lantern floats above the city."
    )

    slow(
        "His ring changes color."
    )

    print()

    print("GREEN")
    print("↓")
    print("YELLOW")

    print()

    slow(
        '"Hal..."'
    )

    slow(
        '"There is someone else inside you."'
    )

    print()

    slow(
        '"There is no Hal Jordan anymore."'
    )

    slow(
        '"Only Parallax."'
    )

    pause()

    # Special boss
    return combat(
        player,
        "PARALLAX-HAL JORDAN",
        850,
        50,
        3000
    )


# ============================================================
# ENDINGS
# ============================================================

def ending(player, victory):

    title("EPILOGUE")

    if not victory:

        print(
            "The emotional spectrum falls into darkness."
        )

        print()

        slow(
            "Your ring goes silent."
        )

        pause()

        return

    ring = player.ring

    if ring == "Omega":
        slow("You are no longer a Green Lantern.")
        slow("You are the founder of the Omega Corp, a white-armored Corps that is completely separate from the White Lanterns.")
        slow("The galaxy will argue forever about whether Kyle saved the spectrum or endangered it.")
        return

    if ring == "Red":

        slow(
            "Your rage nearly consumed you."
        )

        slow(
            "But you discovered something important."
        )

        slow(
            "Rage can be controlled."
        )

    elif ring == "Orange":

        slow(
            "Larfleeze still wants your ring."
        )

        slow(
            "You consider this his highest compliment."
        )

    elif ring == "Yellow":

        slow(
            "Fear tried to control you."
        )

        slow(
            "You chose to control fear instead."
        )

    elif ring == "Green":

        slow(
            "You stand alongside the Green Lantern Corps."
        )

        slow(
            "Your willpower has become legendary."
        )

    elif ring == "Blue":

        slow(
            "Hope spreads throughout the galaxy."
        )

        slow(
            "Saint Walker smiles."
        )

    elif ring == "Indigo":

        slow(
            "You learn that understanding another person"
        )

        slow(
            "can sometimes be stronger than defeating them."
        )

    elif ring == "Violet":

        slow(
            "Love survives the war."
        )

        slow(
            "Carol Ferris welcomes you as an ally."
        )

    elif ring == "Black":

        slow(
            "Death follows wherever you go."
        )

        slow(
            "But you refuse to let it define you."
        )

    elif ring == "White":

        slow(
            "Life returns to worlds that were lost."
        )

        slow(
            "You become a guardian of life itself."
        )

    elif ring == "Ultraviolet":

        slow(
            "You finally confront the emotions you buried."
        )

        slow(
            "The hidden darkness inside you is no longer hidden."
        )

    print()

    line()

    print("FINAL CHARACTER")
    print()

    print(f"Name: {player.name}")
    print(f"Ring: {RINGS[ring]['color']} {ring}")
    print(f"Corps: {player.corps}")
    print(f"Level: {player.level}")

    print()

    # Relationship endings

    print("IMPORTANT RELATIONSHIPS")
    print()

    for person, value in player.relationships.items():

        if value >= 5:
            status = "❤️ Trusted Ally"

        elif value >= 2:
            status = "🙂 Ally"

        elif value <= -3:
            status = "⚔️ Enemy"

        elif value < 0:
            status = "😐 Distrustful"

        else:
            status = "Neutral"

        print(
            f"{person:20} {status}"
        )

    line()

    print()

    slow(
        "THE EMOTIONAL SPECTRUM WILL REMEMBER YOUR NAME."
    )

    pause()


# ============================================================
# SECRET ENDINGS
# ============================================================

def secret_endings(player):

    # White Lantern secret ending
    if player.ring == "White" and player.level >= 8:

        title("SECRET ENDING — GUARDIAN OF LIFE")

        slow(
            "Your white light reaches across the universe."
        )

        slow(
            "Worlds destroyed during the war begin returning."
        )

        slow(
            "You have become something more than a Lantern."
        )

        slow(
            "You have become a guardian of life."
        )

        pause()

        return True

    # Black Lantern secret ending
    if player.ring == "Black" and player.relationships["Black Hand"] >= 4:

        title("SECRET ENDING — THE BLACK CROWN")

        slow(
            "Black Hand kneels."
        )

        slow(
            '"You have proven yourself."'
        )

        slow(
            "The Black Lanterns offer you command."
        )

        choice = ask(
            "Do you accept?",
            [
                "Become their leader.",
                "Destroy the Black Lantern Corps."
            ]
        )

        if choice == 1:

            slow(
                "The galaxy falls into darkness."
            )

        else:

            slow(
                "You turn the Black light against itself."
            )

        pause()

        return True

    return False


# ============================================================
# YELLOW ROUTE
# ============================================================

def yellow_route(player):

    title("YELLOW LANTERN ROUTE")

    slow(
        "Your yellow ring begins behaving strangely."
    )

    slow(
        "You hear Hal Jordan's voice."
    )

    print()

    slow(
        "Don't trust Sinestro."
    )

    slow(
        "He's using you."
    )

    pause()

    sinestro(player)

    title("THE TRUTH")

    slow(
        "You discover that Parallax has been searching"
    )

    slow(
        "for a new host."
    )

    slow(
        "And Hal Jordan has been exposed."
    )

    pause()

    # Special choice
    choice = ask(
        "What will you do?",
        [
            "Save Hal Jordan.",
            "Destroy Parallax.",
            "Join Parallax."
        ]
    )

    if choice == 1:

        player.story_flags.add("save_hal")

        print(
            "\nYou swear that Hal will not be lost."
        )

    elif choice == 2:

        player.story_flags.add("destroy_parallax")

        print(
            "\nYou swear to destroy Parallax."
        )

    else:

        player.story_flags.add("join_parallax")

        print(
            "\nYou hear Parallax whisper..."
        )

    pause()

    victory = final_parallax_hal(player)

    if victory:

        title("YELLOW LANTERN ENDING")

        if "join_parallax" in player.story_flags:

            slow(
                "You don't destroy Parallax."
            )

            slow(
                "You become its chosen champion."
            )

            print()
            print("👁️ FEAR HAS A NEW LANTERN.")

        elif "save_hal" in player.story_flags:

            slow(
                "You reach through the fear."
            )

            slow(
                "Hal Jordan hears your voice."
            )

            slow(
                '"I remember who I am."'
            )

            slow(
                "Parallax is driven from him."
            )

            print()
            print("💚 HAL JORDAN SURVIVES.")

        else:

            slow(
                "Parallax falls."
            )

            slow(
                "Hal Jordan is freed."
            )

            print()
            print("🟡 YOU DEFEATED PARALLAX.")

        pause()

    return victory


# ============================================================
# PLANET-TO-PLANET CORPS ENCOUNTERS
# ============================================================

def planet_to_planet_corps_encounters(player, planet):
    if planet in getattr(player, "visited_planets", set()):
        return
    player.visited_planets.add(planet)

    encounters = [
        ("Red", "Atrocitus"), ("Orange", "Larfleeze"),
        ("Yellow", "Sinestro"), ("Green", "Hal Jordan"),
        ("Blue", "Saint Walker"), ("Indigo", "Indigo-1"),
        ("Violet", "Carol Ferris"), ("Black", "Black Hand"),
        ("Ultraviolet", "Ultraviolet Lantern")]
    print(f"\n🌌 You travel to {planet}.")
    for ring, name in encounters:
        if ring == "Orange" and player.orange_lantern:
            continue
        if ring in getattr(player, "stolen_rings", set()):
            continue
        enemy = {"name": name, "ring": ring,
                 "hp": RINGS.get(ring, {}).get("hp", 200),
                 "max_hp": RINGS.get(ring, {}).get("hp", 200),
                 "attack": RINGS.get(ring, {}).get("attack", 35)}
        choice = ask(f"You encounter {name}, a {ring} Lantern.", ["Talk", "Fight", "Leave"])
        if choice == 1:
            print(f"\nYou talk with {name}.")
            pause()
        elif choice == 2:
            combat(player, name, enemy["hp"], enemy["attack"], 500)
        else:
            print(f"\nYou leave {name} behind.")
            pause()


# ============================================================
# STORY ROUTE SELECTOR
# ============================================================

def choose_story_route(player):
    title("CHOOSE YOUR STORY")

    print("You have created your Lantern. Now choose the story you want to play.")
    print()

    options = [
        "WAR OF LIGHT — The original story",
        "BLACKEST DAY — The dead rise and the emotional spectrum falls into darkness"
    ]

    if player.kyle_mode:
        options.append("OMEGA CORP — Kyle creates his own Corps and every Lantern Corps turns against him")

    choice = ask("Which storyline do you want?", options)

    if choice == 1:
        player.story_route = "war_of_light"
    elif choice == 2:
        player.story_route = "blackest_day"
    else:
        player.story_route = "omega_corp"

    title("STORY SELECTED")

    if player.story_route == "war_of_light":
        print("🟢 WAR OF LIGHT")
        slow("The original Lantern war storyline will continue.")
    elif player.story_route == "blackest_day":
        print("⚫ BLACKEST DAY")
        slow("The dead are rising, Black Lantern rings are spreading, and the light of life is in danger.")
        slow("This route is a game-original adaptation inspired by DC's BLACKEST NIGHT storyline.")
    else:
        print("💚 KYLE — OMEGA CORP")
        slow("You are no longer willing to serve one Corps.")
        slow("You will build your own Corps — the OMEGA CORP.")
        slow("And every Lantern Corps is about to decide that you are their greatest threat.")

    pause()
    return player.story_route


# ============================================================
# BLACKEST DAY STORYLINE
# ============================================================

def blackest_day_intro(player):
    title("BLACKEST DAY — THE DEAD RISE")

    slow("Oa goes silent.")
    slow("Across the galaxy, Lantern rings begin detecting the impossible.")
    slow("The dead are returning.")
    slow("Black rings appear from the darkness and begin searching for the hearts of the living.")
    slow("Black Hand is no longer hiding. He is preparing the universe for a night without light.")
    print()
    slow('A message appears across your ring: "THE DEAD SHALL RISE."')
    pause()


def blackest_day_oa(player):
    title("BLACKEST DAY — OA UNDER ATTACK")

    slow("You return to Oa and see black rings tearing through the Lantern Crypt.")
    slow("The dead Lanterns rise and attack the living Corps.")
    slow("Green Lanterns, Sinestro Corps members, Red Lanterns, Blue Lanterns and the other Corps are forced to fight side by side.")
    print()

    choice = ask(
        "What do you do?",
        [
            "Help defend Oa.",
            "Search the Lantern Crypt for the source.",
            "Talk to the Corps leaders and unite them.",
            "Leave Oa and find Black Hand."
        ]
    )

    if choice == 1:
        combat(player, "Black Lantern Horde", 220, 32, 200)
    elif choice == 2:
        title("THE LANTERN CRYPT")
        slow("You descend into the crypt.")
        slow("Black rings are moving through the tombs like living shadows.")
        slow("You discover that the attacks are being coordinated from beyond the emotional spectrum.")
        player.story_flags.add("blackest_day_source")
        pause()
    elif choice == 3:
        title("THE CORPS UNITED")
        slow("You tell the Lantern leaders that old grudges do not matter if the dead consume everyone.")
        player.relationship("Guardians", 2)
        player.story_flags.add("corps_united")
        pause()
    else:
        title("HUNTING BLACK HAND")
        slow("You leave Oa before the battle can swallow you.")
        slow("Your ring locks onto Black Hand's energy signature.")
        player.story_flags.add("hunting_black_hand")
        pause()

    return player.hp > 0


def blackest_day_black_hand(player):
    title("THE HAND OF DEATH")

    slow("Black Hand appears above a field of black rings.")
    slow('"Every dead soul is a soldier."')
    slow('"Every living heart is a future Black Lantern."')
    print()

    choice = ask(
        "How do you face Black Hand?",
        [
            "Talk to him.",
            "Fight him.",
            "Try to reach the people being controlled by the black rings.",
            "Leave and search for the source of the Black Lantern power."
        ]
    )

    if choice == 1:
        slow("Black Hand laughs, but admits that something even older is moving behind the Black Lanterns.")
        player.story_flags.add("questioned_black_hand")
        player.relationship("Black Hand", -1)
        pause()
        return True
    elif choice == 3:
        slow("You focus on the living instead of the darkness.")
        slow("For a moment, several Black Lanterns hesitate.")
        player.hope += 2
        player.story_flags.add("saved_black_lantern_victims")
        pause()
        return True
    elif choice == 4:
        slow("You refuse to play Black Hand's game.")
        slow("You follow the black energy toward its true source.")
        player.story_flags.add("found_black_source")
        pause()
        return True

    return combat(player, "Black Hand", 400, 38, 750)


def blackest_day_finale(player):
    title("BLACKEST DAY — THE FINAL LIGHT")

    slow("The sky disappears beneath a sea of black rings.")
    slow("The emotional spectrum begins to collapse.")
    slow("Every color of light is being pulled toward the darkness.")
    slow("You realize the battle cannot be won by one Corps alone.")
    print()

    choice = ask(
        "How will you end the Blackest Day?",
        [
            "Unite every Lantern Corps.",
            "Use the White Light of Life.",
            "Destroy the Black Lantern source myself.",
            "Risk everything and enter the darkness."
        ]
    )

    if choice == 1:
        slow("You call on every Corps to fight together.")
        slow("The emotional spectrum answers with a burst of light.")
        player.story_flags.add("united_spectrum")
        pause()
        return True
    elif choice == 2:
        player.white_lantern = True
        player.owned_rings.add("White")
        player.story_flags.add("white_light")
        slow("The White Light of Life answers you.")
        slow("Black rings fall from the sky and shatter.")
        pause()
        return True
    elif choice == 3:
        slow("You attack the source directly.")
        if player.kyle_mode:
            slow("Kyle's 10,000,000-damage strike tears through the darkness.")
            pause()
            return True
        return combat(player, "Avatar of the Black Lantern Source", 650, 42, 2000)
    else:
        slow("You enter the darkness itself.")
        slow("For one impossible moment, you can hear every lost soul in the universe.")
        player.story_flags.add("entered_darkness")
        pause()
        return True


def blackest_day_story(player):
    blackest_day_intro(player)

    if not blackest_day_oa(player):
        return False

    # Give the player a traveling investigation before the finale.
    title("THE DEAD ACROSS THE STARS")
    slow("You travel between worlds while black rings infect the galaxy.")
    slow("Every stop reveals another battle between life and death.")
    pause()

    for planet, enemy in [
        ("Oa", "Black Lantern Guardian"),
        ("Earth", "Black Lantern Hero"),
        ("Ysmault", "Black Lantern Red Lantern"),
        ("Odym", "Black Lantern Blue Lantern")
    ]:
        if player.hp <= 0:
            return False
        print(f"\n🌌 You travel to {planet}.")
        choice = ask(
            f"A Black Lantern threat appears on {planet}. What do you do?",
            ["Talk", "Fight", "Investigate and move on"]
        )
        if choice == 1:
            slow(f"You speak to {enemy} long enough to learn that the black rings are being directed from one central source.")
            player.story_flags.add("heard_black_source")
            pause()
        elif choice == 2:
            if not combat(player, enemy, 190, 30, 175):
                return False
        else:
            slow("You investigate the battlefield instead of fighting.")
            player.story_flags.add("investigated_black_battle")
            pause()

    if not blackest_day_black_hand(player):
        return False

    return blackest_day_finale(player)


# ============================================================
# KYLE — OMEGA CORP STORYLINE
# ============================================================

def omega_corp_foundation(player):
    title("KYLE — THE BIRTH OF OMEGA CORP")

    player.omega_corp = True

    slow("Kyle looks at the Green ring on his hand.")
    slow("It represents willpower, discipline, and the Corps that first gave him a place among the stars.")
    slow("But after everything he has seen, Kyle no longer believes one color should decide the future of the universe.")
    slow("He removes the Green Lantern ring.")
    print()
    print("💚 GREEN LANTERN")
    print("        ↓")
    print("   ✋ RING REMOVED")
    print("        ↓")
    print("⚪ OMEGA LIGHT")
    print()
    slow("The Green light fades from Kyle's armor.")
    slow("A completely different white light surrounds him.")
    slow("It is not the White Lantern Corps. It is not the White Light of Life.")
    slow("It is a new color identity created for one purpose: the Omega Corp.")
    slow("White becomes the color of Omega Corp — a symbol of all the lights forced into one banner.")

    player.ring = "Omega"
    player.corps = "Omega Corp"
    player.omega_corp = True
    player.owned_rings.discard("Green")
    player.owned_rings.add("Omega")
    player.max_hp = 1_000_000_000
    player.hp = player.max_hp
    player.max_energy = 10**18
    player.energy = player.max_energy
    player.attack = 10_000_000
    player.defense = 1_000_000
    player.story_flags.add("kyle_left_green")
    player.story_flags.add("omega_founded")

    print()
    print("⚪⚡ OMEGA CORP ⚡⚪")
    slow("The Omega symbol appears across your armor.")
    slow("You are no longer a Green Lantern.")
    slow("You are the founder and first member of the Omega Corp.")
    pause()


def omega_recruitment(player):
    title("OMEGA CORP — BUILD YOUR CORPS")

    slow("Creating the Omega Corp is easy.")
    slow("Convincing people that they should abandon the old Corps is not.")
    slow("For the first time, Kyle has to lead without the authority of Oa.")
    print()

    candidates = [
        "Rogue Green Lanterns",
        "Disillusioned members of the Sinestro Corps",
        "Blue Lantern healers",
        "Indigo peacekeepers",
        "Star Sapphires who reject the war",
        "Free agents from outside the Corps",
        "Former Red Lantern survivors",
        "Lantern scientists who study spectrum energy"
    ]

    for candidate in candidates:
        choice = ask(
            f"A group of {candidate.lower()} asks to join Omega Corp. What do you do?",
            ["Accept them", "Test their loyalty", "Reject them", "Hear their reasons first"]
        )
        if choice == 1:
            player.omega_members.append(candidate)
            slow(f"They join the Omega Corp. ({len(player.omega_members)} groups recruited.)")
        elif choice == 2:
            slow("You test them by asking what they would do if every old Corps ordered them to leave Omega Corp.")
            answer = ask("Their answer matters.", [
                "They say Omega Corp comes first.",
                "They say they would protect innocent life first.",
                "They refuse to answer.",
            ])
            if answer in (1, 2):
                player.omega_members.append(candidate)
                slow("Kyle accepts their answer and gives them the white Omega emblem.")
            else:
                slow("Kyle refuses to gamble with the future of the new Corps.")
        elif choice == 4:
            slow(f"The {candidate.lower()} explain that they are tired of being treated as tools in someone else's war.")
            reason = ask("Kyle's response:", [
                "Give them a place in Omega Corp.",
                "Tell them power always comes with responsibility.",
                "Tell them they must choose for themselves."
            ])
            if reason in (1, 3):
                player.omega_members.append(candidate)
                slow("They accept the Omega oath.")
            else:
                slow("They leave, but they promise to remember what Kyle said.")
        else:
            slow("You refuse to compromise the new Corps just to make it larger.")
        pause()

    title("THE OMEGA CITADEL")
    slow("With your first recruits behind you, Omega Corp establishes a headquarters beyond the normal Corps territories.")
    slow("White Omega light forms walls, training chambers, communications towers, and a central spectrum chamber.")
    slow("There is no Guardian council here.")
    slow("There is no single emotion here.")
    slow("Every member carries a white Omega insignia, but none of them are White Lanterns.")
    player.story_flags.add("omega_citadel")
    pause()

    title("THE FIRST TEST")
    slow("Kyle gives his new Corps its first order: rescue a convoy trapped between rival Lantern patrols.")
    slow("Instead of choosing a side, Omega Corp enters the battlefield and protects everyone caught in the crossfire.")
    slow("The mission succeeds.")
    slow("But the message sent across the galaxy is interpreted very differently by the old Corps.")
    slow('One transmission repeats again and again: "KYLE IS BUILDING AN ARMY."')
    player.story_flags.add("omega_saved_convoy")
    pause()



def omega_leader_confrontation(player, ring, leader, hp, attack):
    title(f"OMEGA WAR — {leader.upper()}")

    slow(f"{leader} arrives representing the {RINGS[ring]['corps']}.")
    slow(f'"Kyle, the Omega Corp ends today."')
    slow(f'"Every Corps has agreed: you are too powerful to be allowed to continue."')
    print()

    # Combat remains a choice: talk, fight, persuade, or leave.
    return combat(player, leader, hp, attack, 1000)


def omega_final_battle(player):
    title("OMEGA CORP — ALL LIGHTS AGAINST KYLE")

    slow("Your worst prediction comes true.")
    slow("Your white armor is NOT White Lantern armor.")
    slow("It is the unique white color of Omega Corp, created to represent the forced unity of the spectrum.")
    slow("The leaders of every Corps arrive together.")
    slow("Atrocitus. Larfleeze. Sinestro. Hal Jordan. Saint Walker. Indigo-1. Carol Ferris. Black Hand. The Ultraviolet leader.")
    slow("They do not come to negotiate.")
    slow("They come to end Omega Corp before it can reshape the emotional spectrum.")
    print()

    choice = ask(
        "The leaders surround you. What does Kyle do?",
        [
            "Talk to all of them.",
            "Fight every leader.",
            "Offer them one final chance to join Omega Corp.",
            "Unleash the full Omega power of the emotional spectrum."
        ]
    )

    if choice == 1:
        title("THE OMEGA SPEECH")
        slow("You tell the leaders that Omega Corp was never created to destroy them.")
        slow("It was created because the Corps kept turning their differences into wars.")
        slow("For the first time, every leader is silent.")
        player.story_flags.add("omega_speech")
        pause()
        return True

    if choice == 3:
        title("THE LAST OFFER")
        slow("Kyle extends his hand.")
        slow('"Join me, or keep fighting each other forever."')
        slow("Some refuse. Some hesitate.")
        player.story_flags.add("omega_last_offer")
        pause()
        return True

    if choice == 4:
        title("OMEGA LIGHT")
        slow("Every ring color answers Kyle's command.")
        slow("The entire emotional spectrum erupts around you.")
        slow("No Lantern has ever seen anything like it.")
        player.story_flags.add("omega_light")
        pause()
        return True

    # The battle choice is deliberately huge but still uses the normal
    # talk/fight/persuade/leave system for each leader.
    leaders = [
        ("Red", "Atrocitus", 500, 40),
        ("Orange", "Larfleeze", 500, 40),
        ("Yellow", "Sinestro", 500, 40),
        ("Green", "Hal Jordan", 550, 42),
        ("Blue", "Saint Walker", 450, 35),
        ("Indigo", "Indigo-1", 450, 36),
        ("Violet", "Carol Ferris", 475, 38),
        ("Black", "Black Hand", 525, 43),
        ("Ultraviolet", "Ultraviolet Leader", 500, 41),
    ]

    for ring, leader, hp, attack in leaders:
        if player.hp <= 0:
            return False
        if not omega_leader_confrontation(player, ring, leader, hp, attack):
            return False

    return True


def omega_investigation(player):
    title("OMEGA CORP — THE RUMORS SPREAD")

    slow("The new Corps has existed for only a short time, yet the galaxy is already divided over your existence.")
    slow("Some worlds call Omega Corp a peacekeeping force.")
    slow("Others call it the most dangerous Lantern movement ever created.")
    slow("Then the attacks begin.")
    print()

    planets = [
        ("Oa", "Green Lantern surveillance squads"),
        ("Korugar", "Sinestro Corps hunters"),
        ("Odym", "Blue Lantern scouts"),
        ("Ysmault", "Red Lantern raiders"),
        ("Zamaron", "Star Sapphire interceptors")
    ]

    for planet, attackers in planets:
        if player.hp <= 0:
            return False
        title(f"OMEGA CORP — {planet.upper()}")
        slow(f"Omega sensors detect {attackers} moving against you.")
        slow("They do not attack immediately.")
        slow("They demand that Kyle surrender the Omega insignia and dissolve the Corps.")
        choice = ask("How do you respond?", [
            "Open negotiations.",
            "Refuse and prepare Omega Corp defenses.",
            "Investigate who ordered the attack.",
            "Confront their leader directly."
        ])
        if choice == 1:
            slow("You explain that Omega Corp was created to stop Corps from constantly turning against each other.")
            slow("The opposing lanterns do not trust you, but several admit that your convoy rescue was real.")
            player.story_flags.add("omega_negotiated")
        elif choice == 2:
            slow("Omega Corp forms a defensive wall instead of attacking first.")
            slow("The attackers realize you could destroy them, but choose not to.")
            player.story_flags.add("omega_defended")
        elif choice == 3:
            slow("Your investigation reveals the same answer every time: the order is coming from the highest levels of the Corps.")
            slow("Someone wants the old Corps to stop competing with each other long enough to erase Omega Corp.")
            player.story_flags.add("omega_discovered_plot")
        else:
            slow("Kyle flies directly to the opposing commander.")
            slow("Instead of fighting, you demand to know why all the Corps have suddenly united against you.")
            slow('The answer is chilling: "Because you have become a symbol they cannot control."')
            player.story_flags.add("omega_confronted_leaders")
        pause()

    return player.hp > 0


def omega_inner_circle(player):
    title("OMEGA CORP — THE CHOICE OF LEADERSHIP")

    slow("Back at the Omega Citadel, your recruits begin arguing about what the Corps should become.")
    slow("Some want Omega Corp to replace the old Lanterns.")
    slow("Others want Omega Corp to exist only as a peacekeeping force.")
    slow("Kyle realizes the same question that broke the old Corps could break his new one: how much power should one leader have?")
    print()

    choices = [
        "Keep absolute control of Omega Corp.",
        "Create an Omega Council from your recruits.",
        "Promise that Omega Corp will never conquer a world.",
        "Tell everyone the old Corps are still allowed to exist."
    ]
    choice = ask("What kind of leader will Kyle be?", choices)

    if choice == 1:
        slow("You keep the final decision for yourself.")
        player.story_flags.add("omega_absolute_rule")
    elif choice == 2:
        slow("You create an Omega Council so no single emotion can control the Corps.")
        player.story_flags.add("omega_council")
    elif choice == 3:
        slow("You swear Omega Corp will never become another empire.")
        player.story_flags.add("omega_no_conquest")
    else:
        slow("You admit that Omega Corp does not need to destroy the old Corps to prove it belongs in the universe.")
        player.story_flags.add("omega_alliance_offer")

    pause()
    return True


def omega_betrayal_warning(player):
    title("OMEGA CORP — THE WARNING")

    slow("The Citadel alarms suddenly activate.")
    slow("Every Omega recruit looks toward the same screen.")
    slow("Nine symbols appear at once.")
    print()
    print("🔴  🟠  🟡  🟢  🔵  🟣  💗  ⚫  🟪")
    print()
    slow("Every major Lantern Corps has mobilized.")
    slow("They have stopped fighting one another.")
    slow("They have agreed that Kyle is now the greater threat.")
    slow("Your recruits ask what they should do.")

    choice = ask("What order does Kyle give?", [
        "Defend the Citadel but do not attack first.",
        "Move everyone to civilian worlds and protect them.",
        "Meet the Corps leaders alone.",
        "Prepare Omega Corp for the biggest battle in Lantern history."
    ])

    if choice == 1:
        slow("Omega Corp locks down every defensive construct.")
        player.story_flags.add("omega_defensive")
    elif choice == 2:
        slow("You send your recruits away from the coming battle so innocent people will not be caught in it.")
        player.story_flags.add("omega_protected_civilians")
    elif choice == 3:
        slow("Kyle tells the others to stand down.")
        slow("If the Corps leaders want you, they will have to face you themselves.")
        player.story_flags.add("omega_solo_stand")
    else:
        slow("The entire Omega Corp prepares for war.")
        player.story_flags.add("omega_war_ready")

    pause()
    return True


def omega_corp_story(player):
    omega_corp_foundation(player)
    omega_recruitment(player)

    if not omega_investigation(player):
        return False
    if not omega_inner_circle(player):
        return False
    if not omega_betrayal_warning(player):
        return False

    title("THE CORPS DECLARE WAR")
    slow("Your worst prediction comes true.")
    slow("The Lantern Corps stop fighting one another long enough to agree on one thing.")
    slow("Kyle must be stopped.")
    slow("Atrocitus. Larfleeze. Sinestro. Hal Jordan. Saint Walker. Indigo-1. Carol Ferris. Black Hand. The Ultraviolet leader.")
    slow("Every leader receives the same order: destroy the founder of Omega Corp before his new movement becomes unstoppable.")
    slow("This is no longer a dispute between Corps.")
    slow("It is a war against Kyle personally.")
    pause()

    return omega_final_battle(player)


# ============================================================
# ROUTE ENDING
# ============================================================

def route_ending(player, victory):
    if not victory:
        ending(player, False)
        return

    if player.story_route == "blackest_day":
        title("BLACKEST DAY — EPILOGUE")
        slow("The darkness retreats.")
        slow("The dead finally return to silence, and the surviving Corps begin rebuilding together.")
        slow("Your name becomes a symbol of the moment life defeated death.")
        pause()
        ending(player, True)
        return

    if player.story_route == "omega_corp":
        title("OMEGA CORP — EPILOGUE")
        slow("The leaders of the emotional spectrum lower their weapons.")
        if "omega_speech" in player.story_flags or "omega_last_offer" in player.story_flags:
            slow("The galaxy does not see Kyle as a destroyer anymore.")
            slow("It sees the founder of a Corps that challenged every Lantern to change.")
        else:
            slow("The Omega Corp stands victorious beneath a sky filled with every color of light.")
        print()
        print("⚪⚡ OMEGA CORP HAS RISEN ⚡⚪")
        print("Color: WHITE — but this is NOT the White Lantern Corps")
        print("Corps: Omega Corp")
        print(f"Members recruited: {len(player.omega_members)}")
        pause()
        return
        return

    ending(player, True)


# ============================================================
# MAIN STORY
# ============================================================

def main():

    title("💍 EMOTIONAL SPECTRUM")

    print()
    print("             WAR OF LIGHT")
    print()
    print("               RPG v2.0")
    print()

    print(
        "A story about power, emotion, choice, "
        "and the Lantern Corps."
    )

    print()

    name = input("Enter your Lantern's name: ")

    if not name.strip():
        name = "Unknown Lantern"

    player = Player(name)

    if player.kyle_mode:
        print()
        print("💚" * 35)
        print("SECRET CHARACTER UNLOCKED: KYLE")
        print("GREEN LANTERN — UNLIMITED ENERGY")
        print("ATTACK DAMAGE — 10,000,000")
        print("💚" * 35)
        pause()

    # Choose the storyline immediately after naming the character.
    choose_story_route(player)

    # --------------------------------------------------------
    # ROUTE 1 — ORIGINAL WAR OF LIGHT
    # --------------------------------------------------------
    if player.story_route == "war_of_light":

        chapter_one(player)
        chapter_two(player)
        story_interlude(player)

        if not chapter_three(player):
            route_ending(player, False)
            return

        chapter_four(player)
        emotional_evolution(player)

        if not chapter_five(player):
            route_ending(player, False)
            return

        if player.ring == "Yellow":
            victory = yellow_route(player)
        else:
            victory = final_parallax(player)

        if victory:
            if not secret_endings(player):
                route_ending(player, True)
        else:
            route_ending(player, False)

    # --------------------------------------------------------
    # ROUTE 2 — BLACKEST DAY
    # --------------------------------------------------------
    elif player.story_route == "blackest_day":
        victory = blackest_day_story(player)
        route_ending(player, victory)

    # --------------------------------------------------------
    # ROUTE 3 — KYLE / OMEGA CORP
    # --------------------------------------------------------
    elif player.story_route == "omega_corp":
        victory = omega_corp_story(player)
        route_ending(player, victory)


# ============================================================
# START GAME
# ============================================================

if __name__ == "__main__":
    main()
