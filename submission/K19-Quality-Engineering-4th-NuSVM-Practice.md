# K19-Quality-Engineering-NuSVM-Practice

日付 19/04/20

## 課題１：多重割り込みのシステム

### Model code

```bash
​```bash
MODULE main

VAR
    DOWN_FLAG :boolean;
    TIMER_A : 1..9;
    TIMER_B : 1..7;
    TIMER_C : 1..5;

ASSIGN
    init(TIMER_A) :=2;
    init(TIMER_B) :=4;
    init(TIMER_C) :=1;

    next(TIMER_A) :=case
        !(TIMER_A = 9) : TIMER_A + 1;
        TIMER_A = 9 : 1;
    esac;

    next(TIMER_B) :=case
        !(TIMER_B = 7) : TIMER_B + 1;
        TIMER_B = 7 : 1;
    esac;

    next(TIMER_C) :=case
        !(TIMER_C = 5) : TIMER_C + 1;
        TIMER_C = 5 : 1;
    esac;

    init(DOWN_FLAG) := FALSE;
    next(DOWN_FLAG) :=case
        TIMER_A = 9 & TIMER_B = 7 & TIMER_C = 5: TRUE;
        TRUE : FALSE;
    esac;

    SPEC !(EF(DOWN_FLAG) = TRUE)

```

### Evidence

```bash

#============================================================
#DateTime : 20 April 2019 16:16:04
#CmdLine  : NuSMV.exe C:\Users\WASEDA\workspace\svm\svm_exec.smv
#============================================================
*** This is NuSMV 2.6.0 (compiled on Wed Oct 14 15:37:51 2015)
*** Enabled addons are: compass
*** For more information on NuSMV see <http://nusmv.fbk.eu>
*** or email to <nusmv-users@list.fbk.eu>.
*** Please report bugs to <Please report bugs to <nusmv-users@fbk.eu>>

*** Copyright (c) 2010-2014, Fondazione Bruno Kessler

*** This version of NuSMV is linked to the CUDD library version 2.4.1
*** Copyright (c) 1995-2004, Regents of the University of Colorado

*** This version of NuSMV is linked to the MiniSat SAT solver. 
*** See http://minisat.se/MiniSat.html
*** Copyright (c) 2003-2006, Niklas Een, Niklas Sorensson
*** Copyright (c) 2007-2010, Niklas Sorensson

-- specification !(EF DOWN_FLAG = TRUE)  is false
-- as demonstrated by the following execution sequence
Trace Description: CTL Counterexample 
Trace Type: Counterexample 
  -> State: 1.1 <-
    DOWN_FLAG = FALSE
    TIMER_A = 2
    TIMER_B = 4
    TIMER_C = 1
  -> State: 1.2 <-
    TIMER_A = 3
    TIMER_B = 5
    TIMER_C = 2
  -> State: 1.3 <-
    TIMER_A = 4
    TIMER_B = 6
    TIMER_C = 3
  -> State: 1.4 <-
    TIMER_A = 5
    TIMER_B = 7
    TIMER_C = 4
  :
  :
-> State: 1.304 <-
    TIMER_A = 8
    TIMER_B = 6
    TIMER_C = 4
  -> State: 1.305 <-
    TIMER_A = 9
    TIMER_B = 7
    TIMER_C = 5
  -> State: 1.306 <-
    DOWN_FLAG = TRUE
    TIMER_A = 1
    TIMER_B = 1
    TIMER_C = 1

終了コード: 0

```

## 課題２：状態遷移表の検査

### Model Code

```bash
MODULE main

VAR
    EVENT : 0..6;
    STATE_A : {sa_1, sa_2, sa_3};
    STATE_B : {sb_1, sb_2, sb_3};
    fg1 : boolean;
    fg2 : boolean;

ASSIGN
    init(EVENT) := 0;
    init(STATE_A) := sa_1;
    init(STATE_B) := sb_1;
    init(fg1) := FALSE;
    init(fg2) := FALSE;

    next(EVENT) := {1, 2, 3, 4, 5, 6};

    next(STATE_A) :=case
        EVENT = 1 & STATE_A = sa_1 : sa_2;
        EVENT = 1 & STATE_A = sa_2 : sa_1;
        EVENT = 1 & STATE_A = sa_3 : sa_1;
        EVENT = 3 & STATE_A = sa_1 : sa_3;
        EVENT = 5 & STATE_A = sa_2 : sa_3;
        EVENT = 5 & STATE_A = sa_3 : sa_2;
        TRUE : STATE_A;
    esac;

    next(STATE_B) :=case
        EVENT = 2 & STATE_B = sb_1 : sb_1;
        EVENT = 2 & STATE_B = sb_2 : sb_3;
        EVENT = 2 & STATE_B = sb_3 : sb_2;
        EVENT = 4 & STATE_B = sb_1 : sb_2;
        EVENT = 6 & STATE_B = sb_2 : sb_1;
        EVENT = 6 & STATE_B = sb_3 : sb_1;
        TRUE : STATE_B;
    esac;

    next(fg1) :=case
        EVENT = 1 & STATE_A = sa_2 : TRUE;
        EVENT = 3 & STATE_A = sa_3 : FALSE;
        EVENT = 5 & STATE_A = sa_3 : FALSE;
        EVENT = 2 & STATE_B = sb_2 : FALSE;
        EVENT = 4 & STATE_B = sb_3 : TRUE;
        EVENT = 6 & STATE_B = sb_3 : TRUE;
        TRUE : fg1;
    esac;

    next(fg2) :=case
        EVENT = 1 & STATE_A = sa_3 : TRUE;
        EVENT = 3 & STATE_A = sa_2 : TRUE;
        EVENT = 5 & STATE_A = sa_1 : FALSE;
        EVENT = 2 & STATE_B = sb_3 : FALSE;
        EVENT = 4 & STATE_B = sb_2 : FALSE;
        EVENT = 6 & STATE_B = sb_1 : TRUE;
        TRUE : fg2;
    esac;

    SPEC AG((fg1 = TRUE & fg2 = TRUE) -> !EF(fg1 = FALSE & fg2 = FALSE))

```



### Evidence

```bash

#============================================================
#DateTime : 20 April 2019 16:21:01
#CmdLine  : NuSMV.exe C:\Users\WASEDA\workspace\svm\svm_practice2.smv
#============================================================
*** This is NuSMV 2.6.0 (compiled on Wed Oct 14 15:37:51 2015)
*** Enabled addons are: compass
*** For more information on NuSMV see <http://nusmv.fbk.eu>
*** or email to <nusmv-users@list.fbk.eu>.
*** Please report bugs to <Please report bugs to <nusmv-users@fbk.eu>>

*** Copyright (c) 2010-2014, Fondazione Bruno Kessler

*** This version of NuSMV is linked to the CUDD library version 2.4.1
*** Copyright (c) 1995-2004, Regents of the University of Colorado

*** This version of NuSMV is linked to the MiniSat SAT solver. 
*** See http://minisat.se/MiniSat.html
*** Copyright (c) 2003-2006, Niklas Een, Niklas Sorensson
*** Copyright (c) 2007-2010, Niklas Sorensson

-- specification AG ((fg1 = TRUE & fg2 = TRUE) -> !(EF (fg1 = FALSE & fg2 = FALSE)))  is false
-- as demonstrated by the following execution sequence
Trace Description: CTL Counterexample 
Trace Type: Counterexample 
  -> State: 1.1 <-
    EVENT = 0
    STATE_A = sa_1
    STATE_B = sb_1
    fg1 = FALSE
    fg2 = FALSE
  -> State: 1.2 <-
    EVENT = 1
  -> State: 1.3 <-
    EVENT = 3
    STATE_A = sa_2
  -> State: 1.4 <-
    EVENT = 1
    fg2 = TRUE
  -> State: 1.5 <-
    STATE_A = sa_1
    fg1 = TRUE
  -> State: 1.6 <-
    EVENT = 4
    STATE_A = sa_2
  -> State: 1.7 <-
    EVENT = 2
    STATE_B = sb_2
  -> State: 1.8 <-
    STATE_B = sb_3
    fg1 = FALSE
  -> State: 1.9 <-
    EVENT = 1
    STATE_B = sb_2
    fg2 = FALSE

終了コード: 0

```

## 演習：MEMEと連携するサーバの動作

### Model Code

```bash
MODULE main

VAR
    SYSTEM : boolean;
    INPUT : {x, a, b, c};
    MONITER : {print, not_print};
    BUTTON : boolean;
    MEM : {empty, saved_a, saved_ab, saved_ac, saved_abc};

ASSIGN
    init(SYSTEM) := TRUE;
    init(INPUT) := x;
    init(MONITER) := not_print;
    init(BUTTON) := FALSE;
    init(MEM) := empty;

    next(SYSTEM) := {TRUE, FALSE};
    next(INPUT) := {a, b, c};
    
    next(MONITER) :=case
        MEM = saved_abc & BUTTON = FALSE & MONITER = not_print : print;
        BUTTON = TRUE & MONITER = print : not_print;
        TRUE : MONITER;
    esac;

    next(BUTTON) :=case
        MONITER = print & BUTTON = FALSE : TRUE;
        MONITER = print & BUTTON = TRUE : FALSE;
        TRUE : BUTTON;
    esac;

    next(MEM) :=case
        INPUT = a & MEM = empty & SYSTEM = TRUE  : saved_a;
        INPUT = a & MEM = empty & SYSTEM = FALSE : empty;
        INPUT = b & MEM = empty & SYSTEM = TRUE  : empty;
        INPUT = b & MEM = empty & SYSTEM = FALSE : empty;
        INPUT = c & MEM = empty & SYSTEM = TRUE  : empty;
        INPUT = c & MEM = empty & SYSTEM = FALSE : empty;
        
        INPUT = a & MEM = saved_a & SYSTEM = TRUE  : empty;
        INPUT = a & MEM = saved_a & SYSTEM = FALSE : empty;
        INPUT = b & MEM = saved_a : saved_ab;
        INPUT = c & MEM = saved_a : saved_ac;
        
        INPUT = a & MEM = saved_ab & SYSTEM = TRUE  : saved_ab;
        INPUT = a & MEM = saved_ab & SYSTEM = FALSE : empty;
        INPUT = b & MEM = saved_ab & SYSTEM = TRUE  : saved_ab;
        INPUT = c & MEM = saved_ab & SYSTEM = TRUE  : saved_abc;
        
        INPUT = a & MEM = saved_ac & SYSTEM = TRUE  : saved_ac;
        INPUT = a & MEM = saved_ac & SYSTEM = FALSE : empty;
        INPUT = b & MEM = saved_ac & SYSTEM = TRUE  : saved_abc;
        INPUT = c & MEM = saved_ac & SYSTEM = TRUE  : saved_ac;
        
        INPUT = a & MEM = saved_abc & SYSTEM = TRUE  : saved_abc;
        INPUT = a & MEM = saved_abc & SYSTEM = FALSE : empty;
        INPUT = b & MEM = saved_abc & SYSTEM = TRUE  : saved_abc;
        INPUT = c & MEM = saved_abc & SYSTEM = TRUE  : saved_abc;
        
        TRUE : MEM;
    esac;


SPEC !EF(MEM = saved_a & MONITER = print & BUTTON = TRUE)

```

### Evidence

```bash

#============================================================
#DateTime : 20 April 2019 16:22:51
#CmdLine  : NuSMV.exe C:\Users\WASEDA\workspace\svm\svm_exercise.smv
#============================================================
*** This is NuSMV 2.6.0 (compiled on Wed Oct 14 15:37:51 2015)
*** Enabled addons are: compass
*** For more information on NuSMV see <http://nusmv.fbk.eu>
*** or email to <nusmv-users@list.fbk.eu>.
*** Please report bugs to <Please report bugs to <nusmv-users@fbk.eu>>

*** Copyright (c) 2010-2014, Fondazione Bruno Kessler

*** This version of NuSMV is linked to the CUDD library version 2.4.1
*** Copyright (c) 1995-2004, Regents of the University of Colorado

*** This version of NuSMV is linked to the MiniSat SAT solver. 
*** See http://minisat.se/MiniSat.html
*** Copyright (c) 2003-2006, Niklas Een, Niklas Sorensson
*** Copyright (c) 2007-2010, Niklas Sorensson

-- specification !(EF ((MEM = saved_a & MONITER = print) & BUTTON = TRUE))  is false
-- as demonstrated by the following execution sequence
Trace Description: CTL Counterexample 
Trace Type: Counterexample 
  -> State: 1.1 <-
    SYSTEM = TRUE
    INPUT = x
    MONITER = not_print
    BUTTON = FALSE
    MEM = empty
  -> State: 1.2 <-
    INPUT = a
  -> State: 1.3 <-
    SYSTEM = FALSE
    INPUT = b
    MEM = saved_a
  -> State: 1.4 <-
    SYSTEM = TRUE
    INPUT = c
    MEM = saved_ab
  -> State: 1.5 <-
    SYSTEM = FALSE
    INPUT = a
    MEM = saved_abc
  -> State: 1.6 <-
    SYSTEM = TRUE
    MONITER = print
    MEM = empty
  -> State: 1.7 <-
    SYSTEM = FALSE
    BUTTON = TRUE
    MEM = saved_a

終了コード: 0

```

