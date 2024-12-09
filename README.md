# Blackjack
I made this simple project to create a casino-styled blackjack game between the dealer and the player. The goal is to have the player feel immersed in the program by having a hit/stand mechanic with win rate percentages fluctuating in between hits.

#include <iostream>
#include <vector>
#include <ctime>
#include <cstdlib>
#include <iomanip>  // For controlling decimal precision

using namespace std;

// Card representation
enum Rank { TWO = 2, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN = 10, JACK = 10, QUEEN = 10, KING = 10, ACE = 11 };
enum Suit { HEARTS, DIAMONDS, CLUBS, SPADES };

// Card structure
struct Card {
    Rank rank;
    Suit suit;
};

// Deck of cards
class Deck {
public:
    Deck() {
        for (int suit = 0; suit < 4; ++suit) {
            for (int rank = 2; rank <= 14; ++rank) {
                cards.push_back(Card{static_cast<Rank>(rank), static_cast<Suit>(suit)});
            }
        }
        shuffleDeck();
    }

    void shuffleDeck() {
        srand(time(0)); // Random seed
        for (int i = 0; i < cards.size(); ++i) {
            int j = rand() % cards.size();
            swap(cards[i], cards[j]);
        }
    }

    Card dealCard() {
        Card dealtCard = cards.back();
        cards.pop_back();
        return dealtCard;
    }

    int cardsRemaining() { return cards.size(); }

private:
    vector<Card> cards;
};

// Hand class to represent the player's or dealer's hand
class Hand {
public:
    void addCard(Card card) {
        hand.push_back(card);
    }

    int getHandValue() {
        int value = 0;
        int aceCount = 0;

        // Calculate the total value of the hand
        for (const auto& card : hand) {
            if (card.rank == ACE) {
                value += 11; // Add 11 for Ace initially
                aceCount++;
            }
            else {
                value += (card.rank >= JACK) ? 10 : card.rank; // Treat face cards as 10
            }
        }

        // Adjust for Aces if the total value exceeds 21
        while (value > 21 && aceCount > 0) {
            value -= 10; // Convert one Ace from 11 to 1
            aceCount--;   // Reduce the count of Aces being treated as 11
        }

        return value;
    }

    bool isBusted() { return getHandValue() > 21; }

    void displayHand() {
        for (const auto& card : hand) {
            cout << getCardString(card) << " ";
        }
        cout << "Value: " << getHandValue() << endl;
    }

private:
    vector<Card> hand;

    string getCardString(const Card& card) {
        const string ranks[] = {"2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"};
        const string suits[] = {"Hearts", "Diamonds", "Clubs", "Spades"};
        return ranks[card.rank - 2] + " of " + suits[card.suit];
    }
};

// Function to simulate the probability of the player's hand winning
double calculateWinProbability(Hand playerHand, Hand dealerHand, Deck& deck) {
    int wins = 0;
    int simulations = 10000;

    for (int i = 0; i < simulations; ++i) {
        Deck simulationDeck = deck;
        Hand simulationPlayerHand = playerHand;
        Hand simulationDealerHand = dealerHand;

        while (simulationPlayerHand.getHandValue() < 17 && !simulationPlayerHand.isBusted()) {
            simulationPlayerHand.addCard(simulationDeck.dealCard());
        }

        while (simulationDealerHand.getHandValue() < 17 && !simulationDealerHand.isBusted()) {
            simulationDealerHand.addCard(simulationDeck.dealCard());
        }

        if (simulationPlayerHand.isBusted()) continue;
        if (simulationDealerHand.isBusted()) wins++;
        else if (simulationPlayerHand.getHandValue() > simulationDealerHand.getHandValue()) wins++;
    }

    double baseWinProbability = static_cast<double>(wins) / simulations;

    // Adjust win probability based on how close the player's hand is to 21
    int playerHandValue = playerHand.getHandValue();
    if (playerHandValue > 21) {
        return 0.0;  // Player busted, no chance of winning
    } else {
        double closenessTo21 = (21 - playerHandValue) / 21.0; // Closeness to 21 (0.0 to 1.0)
        double adjustedProbability = baseWinProbability * closenessTo21; // Adjust the win probability
        return adjustedProbability;
    }
}

int main() {
    Deck deck;

    // Create player and dealer hands
    Hand playerHand;
    Hand dealerHand;

    // Deal two cards to both player and dealer
    playerHand.addCard(deck.dealCard());
    playerHand.addCard(deck.dealCard());
    dealerHand.addCard(deck.dealCard());
    dealerHand.addCard(deck.dealCard());

    // Display hands
    cout << "Initial Hands:\n";
    cout << "Dealer Hand: ";
    dealerHand.displayHand();
    cout << "Player Hand: ";
    playerHand.displayHand();

    // Calculate and display win probability before any hits
    double winProbabilityBeforeHit = calculateWinProbability(playerHand, dealerHand, deck);
    cout << "\nWin Probability before hit: " << fixed << setprecision(2) << winProbabilityBeforeHit * 100 << "%" << endl;

    // Simulate player's hits until they either bust or reach 21
    while (playerHand.getHandValue() < 21 && !playerHand.isBusted()) {
        cout << "\nDo you want to hit? (y/n): ";
        char choice;
        cin >> choice;

        if (choice == 'y' || choice == 'Y') {
            playerHand.addCard(deck.dealCard());
            playerHand.displayHand();
        } else {
            break;
        }

        // Check if the player has busted
        if (playerHand.isBusted()) {
            cout << "\nYou have busted! Your hand value is over 21.\n";
            break;
        }

        // Display updated win probability after the player's action
        double winProbabilityAfterHit = calculateWinProbability(playerHand, dealerHand, deck);
        cout << "\nWin Probability after your action: " << fixed << setprecision(2) << winProbabilityAfterHit * 100 << "%" << endl;
    }

    // If the player did not bust, proceed with the dealer's actions
    if (!playerHand.isBusted()) {
        cout << "\nDealer's Turn:\n";
        while (dealerHand.getHandValue() < 17) {
            dealerHand.addCard(deck.dealCard());
            dealerHand.displayHand();
        }

        // Determine the winner
        if (dealerHand.isBusted()) {
            cout << "\nDealer busts! You win!\n";
        } else if (playerHand.getHandValue() > dealerHand.getHandValue()) {
            cout << "\nYou win! Your hand is higher than the dealer's.\n";
        } else if (playerHand.getHandValue() < dealerHand.getHandValue()) {
            cout << "\nDealer wins! Their hand is higher.\n";
        } else {
            cout << "\nIt's a tie!\n";
        }
    }

    return 0;
}
