# Story 5 - fifth feature - leader board for single player games

> As an internet user

> I want to see a leader board

> which will show my high score against everyone else who's played the same game

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
    
