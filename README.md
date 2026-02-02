# shapiro_enzymatic_automaton
A simulation of the Enzymatic Automaton by Shapiro et al.

## Theoretical summary

The core idea here is that we can represent the current state and the next input symbol using 4 bases out of the 6 bases of the actual representation of every symbol, which is exactly the length of the sticky end left by foki.

We synthesize the input sequence on a double stranded dna molecule (aka the "software") and we engineer transition molecules (the "hardware") that help us digest the input according to the automata state transitions. These transition molecules essentially encode triples of information - q_current, in_symbol, q_next. The q_current, in_symbol part is a sticky end which is the complementary dna sequence of the 4-bases of our encoding of the state+input because it is designed to hybridize and ligate to the remaining chunk of the "software", the q_next part essentially contains a foki recoginition site to facilitate the ingestion process and some padding to account for how we transform the 6 bases of each letter to 4 bases using displacements associated with each state - this is the key part, it chops the next letter IN THE INPUT according to the next state's displacement. so if the next state is q1 it chops exactly at the beginning of the next symbol, leaving the first 4 bases as sticky end (followed by the last 2 bases of the symbol paired up), and if the next state is q0 then it chops off the first 2 bases entirely off the input and leaves the remaining 4 bases as a sticky end (followed by the next symbol in the input) and so on.

---

## Reference Data (from Shapiro Paper)

### Symbol Encodings (6 bases each)
| Symbol | Sequence |
|--------|----------|
| a | CTGGCT |
| b | CGCAGC |
| t (terminator) | TGTCGC |

### State-Symbol Sticky Ends (4 bases)
Based on state displacement (Δ):

| State | Δ | 'a' sticky end | 'b' sticky end | 't' sticky end |
|-------|---|----------------|----------------|----------------|
| q0 | 2 | GGCT (last 4) | CAGC (last 4) | TCGC (last 4) |
| q1 | 0 | CTGG (first 4) | CGCA (first 4) | TGTC (first 4) |

### Transition Molecule Structure
Each transition molecule has 3 components (5' to 3' on top strand):

```
[FokI Recognition Site: GGATG] - [Spacer: 1/3/5 bases] - (then complementary sticky end on bottom strand overhang)
```

### Spacer Length Rules (Table 5.6)
| Spacer Length | State Transition |
|---------------|------------------|
| 1 | q1 → q0 |
| 3 | State unchanged (q0→q0 or q1→q1) |
| 5 | q0 → q1 |

### All 8 Transition Molecules (Figure 5.22)
| ID | Transition | Top (5'→3') | Bottom (5'→3') |
|----|------------|-------------|----------------|
| T1 | q0 -a→ q0 | GGATGTAC | CCTACATGCCGA |
| T2 | q0 -a→ q1 | GGATGACGAC | CCTACTGCTGCCGA |
| T3 | q0 -b→ q0 | GGATGACG | CCTACTGCGTCG |
| T4 | q0 -b→ q1 | GGATGACGAC | CCTACTGCTGGTCG |
| T5 | q1 -a→ q0 | GGATGA | CCTACTGACC |
| T6 | q1 -a→ q1 | GGATGACG | CCTACTGCGACC |
| T7 | q1 -b→ q0 | GGATGG | CCTACCGCGT |
| T8 | q1 -b→ q1 | GGATGACG | CCTACTGCGCGT |

### FokI Cutting Positions
- **Top strand (5')**: Cut 9 bases after last base of recognition site
- **Bottom strand (3')**: Cut 13 bases after last base of recognition site
- Creates a 4-base 5' overhang (the new sticky end)
 