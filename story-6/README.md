# Story 6 - sixth feature - multi round game

> As an internet user

> I want to play a game that lasts for longer than 1 round

> which will calculate my score as a total of all rounds' scores for single player games
> which will decide the winner based on number of rounds won for multi player games

* Introduction to all the layers
  * “Lifecycle Flow - two player game, I guess incorrect, I WIN”
    - GIVEN term of "flowers",
    - WHEN I visit the game,
    - AND I select "start new two player game",
    - THEN I see a copyable URL,
    - WHEN I click "Ready",
    - AND my friend visits the URL provided,
    - AND my friend clicks "Ready",
    - THEN I see images of flowers,
    - WHEN I guess the term "flowers",
    - THEN I see a message "WINNER",
    - AND my friend sees a message "LOSER",
    
  * “Lifecycle Flow - two player game, I guess incorrect, they guess correct, they WIN”
    - GIVEN term of "flowers",
    - WHEN I visit the game,
    - AND I select "start new two player game",
    - THEN I see a copyable URL,
    - WHEN I click "Ready",
    - AND my friend visits the URL provided,
    - AND my friend clicks "Ready",
    - THEN I see images of flowers,
    - WHEN I guess the term "roses",
    - THEN I see a message "wrong answer, try again",
    - WHEN my friend guesses the term "flowers",
    - THEN I see a message "LOSER",
    - AND my friend sees a message "WINNER", 
    
  * “Lifecycle Flow - two player game, I guess incorrect, I guess correct, I WIN”
    - GIVEN term of "flowers",
    - WHEN I visit the game,
    - AND I select "start new two player game",
    - THEN I see a copyable URL,
    - WHEN I click "Ready",
    - AND my friend visits the URL provided,
    - AND my friend clicks "Ready",
    - THEN I see images of flowers,
    - WHEN I guess the term "roses",
    - THEN I see a message "wrong answer, try again",
    - WHEN I guess the term "flowers",
    - THEN I see a message "WINNER",
    - AND my friend sees a message "LOSER", 

  ** mechanics
    - same person play same game twice
    - different person play same game
    - higher score than existing
    - lower score than existing
    - same score as existing
    
  ** mechanics
    - these should cover multi round games?