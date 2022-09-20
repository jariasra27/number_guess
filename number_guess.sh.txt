#!/bin/bash
PSQL="psql --username=freecodecamp --dbname=number_guess --tuples-only -c"
#Function check number
Is_Number(){
  if [[ $1 =~ ^[0-9]+$ ]]
  then
    VALID="True"
  else
    VALID="False"
  fi      
}
#Function to guess a number
Guess_Number(){
  OK="False"
  ATTEMPTS=0
  #Generate random number
  NUMBER=$(( RANDOM % 1000 + 1))
  #Ask user to guess a number
  echo "Guess the secret number between 1 and 1000:"
  read NUMBER_GUESS
  #Validate if the number is a integer
  Is_Number $NUMBER_GUESS
  while [[ $VALID = "False" ]]
  do
    echo "That is not an integer, guess again:"
    read NUMBER_GUESS
    Is_Number $NUMBER_GUESS
  done
  #Guess the number
  while [[ $OK = "False" ]]
  do
    ATTEMPTS=$((ATTEMPTS + 1))
    if [[ $NUMBER -lt $NUMBER_GUESS ]]
    then
      echo "It's lower than that, guess again:"
      read NUMBER_GUESS
      Is_Number $NUMBER_GUESS
      while [[ $VALID = "False" ]]
      do
        echo "That is not an integer, guess again:"
        read NUMBER_GUESS
        Is_Number $NUMBER_GUESS
      done
    elif [[ $NUMBER -gt $NUMBER_GUESS ]]
    then
      echo "It's higher than that, guess again:"
      read NUMBER_GUESS
      Is_Number $NUMBER_GUESS
      while [[ $VALID = "False" ]]
      do
        echo "That is not an integer, guess again:"
        read NUMBER_GUESS
        Is_Number $NUMBER_GUESS
      done
    elif [[ $NUMBER_GUESS = $NUMBER ]]
    then
      echo "You guessed it in $ATTEMPTS tries. The secret number was $NUMBER. Nice job!"
      OK="True"
    fi
  done
}
#Ask for the user information
echo "Enter your username:"
read USERNAME
CHECK_USERNAME=$($PSQL "SELECT username, games_played, best_game FROM players WHERE username = '$USERNAME'")
if [[ -z $CHECK_USERNAME ]]
then
  echo "Welcome, $USERNAME! It looks like this is your first time here."
  INSERT_USER=$($PSQL "INSERT INTO players(username,games_played,best_game) VALUES ('$USERNAME',0,0)")
  GAMES_PLAYED=0
  BEST_GAME=0
else
  echo "$CHECK_USERNAME" | while read USERNAME BAR GAMES_PLAYED BAR BEST_GAME
  do
    echo "Welcome back, $USERNAME! You have played $GAMES_PLAYED games, and your best game took $BEST_GAME guesses."
  done
fi
Guess_Number
#Update user game statistics
USERNAME_DATA=$($PSQL "SELECT games_played, best_game FROM players WHERE username = '$USERNAME'")
echo "$USERNAME_DATA" | while read GAMES_PLAYED BAR BEST_GAME
do
  if [[ $GAMES_PLAYED = 0 ]]
  then
    UPDATE=$($PSQL "UPDATE players SET games_played=$GAMES_PLAYED+1, best_game=$ATTEMPTS WHERE username = '$USERNAME'")
  else
    if [[ $BEST_GAME -gt $ATTEMPTS ]]
    then
      UPDATE=$($PSQL "UPDATE players SET games_played=$GAMES_PLAYED+1, best_game=$ATTEMPTS WHERE username = '$USERNAME'")
    else
      UPDATE=$($PSQL "UPDATE players SET games_played=$GAMES_PLAYED+1 WHERE username = '$USERNAME'")
    fi
  fi
done
