---
layout: post
title: texas hold 'em series (4): hand evaluation 
tags: [lua文章]
categories: [topic]
---
  * Hand Values
  * Evaluation
  * Simulation
  * Example

In the previous post, we considered the probabilities of making one specific
hand with the turn/river card. This can be rather useful in specific
situations, but still cannot apply thoughout a game. Poker is essentially an
incomplete information game. Different from Go, where you can see all stones
placed on the chessboard and thereby “solve” an optimal move, you never know
you opponents’ pocket cards until showdown (yet even then, people mucks).
Also, you have little clue on the unshown community cards. Therefore, in order
to evaluate a hand during a poker game, we’d better opt for a online
evaluation algorithm instead of considering this as a DP-like problem.

# Hand Values

For the sake of convenience, the table of hand values is shown again below.

Name | Description | Example  
---|---|---  
High Card | Simple value of the card. Lowest: deuce; highest: ace. | `As` `4s`
`7h` `Td` `2c`  
Pair | Two cards with the same value. | `As` `4s` `7h` `Td` `Ac`  
Two Pairs | Two pairs where each pair of cards have the same value. | `As`
`4s` `4h` `Td` `Ac`  
Three of a Kind | Three cards with the same value. | `As` `4s` `4h` `4d` `2c`  
Straight | Five cards in consecutive values (ace can precede deuce or follow
up king). | `9s` `Ts` `Jh` `Qd` `Kc`  
Flush | Five cards of the same suit. | `Ah` `4h` `7h` `Th` `2h`  
Full House | Three of a kind with the rest two making a pair. | `As` `4s` `4h`
`4d` `Ac`  
Four of a Kind | Four cards of the same value. | `As` `4s` `4h` `4d` `4c`  
Straight Flush | Straight of the same suit. | `9h` `Th` `Jh` `Qh` `Kh`  
Royal Flush | Straight flush from ten to ace. | `Th` `Jh` `Qh` `Kh` `Ah`  
  
# Evaluation

Our ultimate goal is to be able to evaluate the probability of a win (and
tie), i.e. the relative strength of our hand. However, let’s just try to get
one step back and consider evaluating the absolute strengths. Here we denote
it as the **hand value**. Ideally, there are (binom{52}{5}=2,598,960) possible
hands but much less valid values. An intuitive idea to match all these hands
to their values, which is also what I did, is to first generate a sparse
mapping from hands to values, and then condense it. First we need a function
that identifies hand types. Then, for hands within the same type, we encode
them like a carry system (e.g. decimal); for hands across different types, we
manually add offsets so that higher hand types always yield higher values. The
final results are stored in the dictionary `hv` and serialized locally. The
highest value is 6144 and here is a glance of `hv`:

    
    
    {
        '2s3s4s5s7h' : 0,
        '2s3s4s5s7c' : 0,
        '2s3s4s5s7d' : 0,
        '2s3s4s5h7s' : 0,
        '2s3s4s5h7h' : 0,
        ...
        '9dTdJdQdKd' : 6143,
        'TsJsQsKsAs' : 6144,
        'ThJhQhKhAh' : 6144,
        'TcJcQcKcAc' : 6144,
        'TdJdQdKdAd' : 6144,
    }
    

# Simulation

Now that we have the full mapping from hands to values, there are two things
we can do to calculate the probabilities:

  1. Enumerate all opponent hands exhaustively, and compare hand values
  2. Simulate certain number of opponent hands, and then compare

Things are gonna be much easier when we only consider the two-player case
(a.k.a. heads-up games). In that case, we can literally count and evaluate all
scenarios — when there’re 2 pocket cards for me and 3 community cards shown,
we only need to enumerate (binom{52-5}{2}=1,081) opponent hands, which is
lightning fast to finish with modern programming languages like Python or C++.
However, when we have more players, like 5, the first method gets nasty. The
hands of each opponent are not independent, so we have to go through
(binom{52-5}{2times 4} > 3times 10^8) situations and that, different to the
heads-up scenario, would be unacceptably slow no matter which language we use.
Therefore, we opt for the second method at some cost of precision. The code
(partial) is shared below.

    
    
    def handEval(my_pocket, community, n_players, hv,
                 n_tot=1024, ranges=None, verbose=False):
        '''
        PARAMETERS:
            my_pocket: list of 2 strings
            community: list of 3-5 strings
            n_players: integer
            hv: hand_value dictionary
            n_tot: integer
            ranges: list of (n_players - 1) floats (default: None)
            verbose: boolean
    
        RETURN: (float, float)
        '''
        assert len(set(my_pocket + community)) == len(my_pocket + community) and 
               all(card in deck for card in my_pocket + community), 'invalid cards'
        assert n_players < 10, 'too many players'
        n_unshown = 5 - len(community)
        np.random.seed(123)
        if ranges is None: ranges = [1] * (n_players - 1)
        simulation = []
        for j in range(n_tot):
            available = [c for c in deck if (c not in my_pocket + community)]
            sim = np.random.choice(available, n_unshown, replace=False).tolist()
            available = [c for c in available if c not in sim]
            for i in range(n_players - 1):
                pf_range = [pf for k, pf in enumerate(pf_full) if
                            pf[0] in available and pf[1] in available and
                            k < len(pf_full) * ranges[i]]
                idx = np.random.choice(range(len(pf_range)))
                cards = pf_range[idx]
                sim += cards
                available = [c for c in available if c not in cards]
            simulation.append(sim)
            if verbose:
                print('{}/{} ({:.2%})'.format(j + 1, n_tot, (j + 1) / n_tot), end='r')
        n_win = n_tie = 0
        if not n_tot: n_tot = len(simulation)
        for i, cards in enumerate(simulation):
            cards = list(cards)
            new_community = cards[:n_unshown]
            my_full_hand = my_pocket + community + new_community
            op_full_hands = []
            for j in range(n_players - 1):
                op_pocket = cards[n_unshown + j * 2:n_unshown + j * 2 + 2]
                op_full_hands.append(op_pocket + community + new_community)
            my_value = value(my_full_hand, hv)[0]
            op_values = [value(op_full_hand, hv)[0] for op_full_hand in op_full_hands]
            op_value = max(op_values)
            # print(value(my_full_hand, hv)[1])
            # for op_full_hand in op_full_hands:
            #     print(value(op_full_hand, hv)[1])
            # print()
            n_win += (my_value > op_value)
            n_tie += (my_value == op_value)
        if verbose: print(' ' * 50, end='r')
        return n_win / n_tot, n_tie / n_tot
    
    
    if __name__ == '__main__':
        with open('hv.json', 'r') as f: hv = json.load(f)
        my_pocket = input('Pocket: ').split()
        community = input('Community: ').split()
        n_players = int(input('Players: '))
        ranges = input('Opponent Ranges: ').split()
        ranges = [float(i) for i in ranges] if ranges else None
        p_win, p_tie = handEval(my_pocket, community, n_players, hv, ranges=ranges, verbose=True)
        print('P(win) = {:.4%}nP(tie) = {:.4%}'.format(p_win, p_tie))
    

# Example

Below are two example tests.

    
    
    Pocket: Tc Jd
    Community: 4h 5h 6d 2h Kh
    Players: 2
    Opponent Ranges: 
    
    
    
    P(win) = 5.4688%                                  
    P(tie) = 0.4883%
    
    
    
    Pocket: Tc Jd
    Community: 4h 5h 6d 2h Kh
    Players: 2
    Opponent Ranges: 0.1
    
    
    
    P(win) = 1.9531%                                  
    P(tie) = 0.7812%
    

Note here I also implement an interesting parameter called `ranges` which
represents the opponents prior ranges at pre-flop. When passing an empty
value, all combinations of two cards are considered. When we specify a list of
ranges (numbers from 0 to 1, say (x%)), then the opponents are assumed to only
play when their pocket cards are at least in the top (x%) pairs of all pairs.
See this [table](http://holdemtight.com/pgs/od/oddpgs/3-169holdemhands.htm)
for more reasoning.