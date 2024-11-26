# BlackJack12
This is a project for app dev creating blackjack simulator by stewart bowman.

import random

# Card Suits, Ranks, and Values
suits = ('Hearts', 'Diamonds', 'Spades', 'Clubs')
ranks = ('Two', 'Three', 'Four', 'Five', 'Six', 'Seven', 'Eight', 'Nine', 'Ten', 'Jack', 'Queen', 'King', 'Ace')
values = {'Two': 2, 'Three': 3, 'Four': 4, 'Five': 5, 'Six': 6, 'Seven': 7, 'Eight': 8, 'Nine': 9, 'Ten': 10, 
          'Jack': 10, 'Queen': 10, 'King': 10, 'Ace': 11}

# Classes
class Card:
    def __init__(self, suit, rank):
        self.suit = suit
        self.rank = rank

    def __str__(self):
        return f"{self.rank} of {self.suit}"

class Deck:
    def __init__(self):
        self.deck = [Card(suit, rank) for suit in suits for rank in ranks]

    def shuffle(self):
        random.shuffle(self.deck)

    def deal(self):
        return self.deck.pop()

class Hand:
    def __init__(self):
        self.cards = []
        self.value = 0
        self.aces = 0

    def add_card(self, card):
        self.cards.append(card)
        self.value += values[card.rank]
        if card.rank == 'Ace':
            self.aces += 1

    def adjust_for_ace(self):
        while self.value > 21 and self.aces:
            self.value -= 10
            self.aces -= 1

class Chips:
    def __init__(self):
        self.total = 100
        self.bet = 0

    def win_bet(self):
        self.total += self.bet

    def lose_bet(self):
        self.total -= self.bet

# Functions
def ask_to_bet():
    """
    Ask the player if they would like to place a bet before proceeding.
    """
    while True:
        response = input("Would you like to place a bet? (yes/no): ").strip().lower()
        if response == 'yes':
            return True  # Proceed to place a bet
        elif response == 'no':
            print("You must place a bet to play. Please reconsider.")
        else:
            print("Invalid input. Please type 'yes' to place a bet or 'no' to exit.")

def take_bet(chips):
    """
    Prompt the player to place a bet after confirming their willingness.
    """
    while True:
        try:
            print(f"\nYou currently have {chips.total} chips.")
            chips.bet = int(input("How many chips would you like to bet? "))
            if chips.bet <= 0:
                print("Invalid bet! You must bet more than zero.")
            elif chips.bet > chips.total:
                print("Insufficient chips! You cannot bet more than your total chips.")
            else:
                print(f"Bet accepted: {chips.bet} chips.")
                break
        except ValueError:
            print("Invalid input! Please enter a valid number.")

def hit(deck, hand):
    hand.add_card(deck.deal())
    hand.adjust_for_ace()

def split_hand(deck, player_hand):
    # Split the hand if the first two cards are the same
    card1 = player_hand.cards.pop()  # Remove one card from hand
    card2 = player_hand.cards.pop()  # Remove the second card from hand
    hand1 = Hand()  # Create a new hand for the split
    hand2 = Hand()  # Create a second hand for the split
    hand1.add_card(card1)  # Add the first card to the first hand
    hand1.add_card(deck.deal())  # Deal a new card to the first hand
    hand2.add_card(card2)  # Add the second card to the second hand
    hand2.add_card(deck.deal())  # Deal a new card to the second hand
    return hand1, hand2  # Return the two new hands

def double_down(deck, player_hand, chips):
    # Double down, the player doubles their bet and gets only one card
    chips.bet *= 2  # Double the bet
    hit(deck, player_hand)  # Player gets one more card
    print(f"Your bet is now {chips.bet} chips.")
    return player_hand  # Return the updated hand

def hit_or_stand(deck, hand, chips):
    while True:
        print("\nYour current hand:", *hand.cards, sep='\n ')
        print(f"Hand value: {hand.value}")
        choice = input("Would you like to Hit, Stand, Split or Double Down? (hit/stand/split/double down): ").strip().lower()
        if choice == 'hit':
            hit(deck, hand)
            return True  # Continue playing
        elif choice == 'stand':
            print("You have chosen to stand.")
            return False  # Stop playing
        elif choice == 'split' and len(hand.cards) == 2 and values[hand.cards[0].rank] == values[hand.cards[1].rank]:
            # Check if the player has two cards of the same rank to split
            print("You chose to split.")
            hand1, hand2 = split_hand(deck, hand)  # Call the split function
            return [hand1, hand2]  # Return the two new hands after split
        elif choice == 'double down' and len(hand.cards) == 2:
            print("You chose to double down.")
            return double_down(deck, hand, chips)  # Call the double down function
        else:
            print("Invalid input. Please choose 'hit', 'stand', 'split' or 'double down'.")

def show_some(player, dealer):
    print("\nDealer's Hand:")
    print(" <card hidden>")
    print('', dealer.cards[1])
    print("\nPlayer's Hand:", *player.cards, sep='\n ')

def show_all(player, dealer):
    print("\nDealer's Hand:", *dealer.cards, sep='\n ')
    print("Dealer's Hand =", dealer.value)
    print("\nPlayer's Hand:", *player.cards, sep='\n ')
    print("Player's Hand =", player.value)

def player_busts(chips):
    print("Player busts!")
    chips.lose_bet()

def player_wins(chips):
    print("Player wins!")
    chips.win_bet()

def dealer_busts(chips):
    print("Dealer busts!")
    chips.win_bet()

def dealer_wins(chips):
    print("Dealer wins!")
    chips.lose_bet()

def push():
    print("It's a tie! Push.")

# Main Game Loop
while True:
    print("Welcome to Blackjack! Try to get as close to 21 as possible without going over.\n")

    deck = Deck()
    deck.shuffle()

    player_hand = Hand()
    dealer_hand = Hand()

    player_hand.add_card(deck.deal())
    player_hand.add_card(deck.deal())
    dealer_hand.add_card(deck.deal())
    dealer_hand.add_card(deck.deal())

    player_chips = Chips()

    # Ask if the player wants to place a bet
    if ask_to_bet():
        take_bet(player_chips)

    show_some(player_hand, dealer_hand)

    playing = True
    while playing:
        if isinstance(player_hand, list):  # Case for split hands
            for hand in player_hand:
                playing = hit_or_stand(deck, hand, player_chips)
                if hand.value > 21:
                    player_busts(player_chips)
        else:
            playing = hit_or_stand(deck, player_hand, player_chips)
            if player_hand.value > 21:
                player_busts(player_chips)
                break

    if player_hand.value <= 21:
        while dealer_hand.value < 17:
            hit(deck, dealer_hand)
        show_all(player_hand, dealer_hand)

        if dealer_hand.value > 21:
            dealer_busts(player_chips)
        elif dealer_hand.value > player_hand.value:
            dealer_wins(player_chips)
        elif dealer_hand.value < player_hand.value:
            player_wins(player_chips)
        else:
            push()

    print(f"\nPlayer's total chips: {player_chips.total}")
    new_game = input("Would you like to play again? (y/n): ").strip().lower()
    if new_game != 'y':
        print("Thanks for playing! Goodbye!")
        break







