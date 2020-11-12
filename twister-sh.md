Sometimes, you just need to wash the dishes even though the kids are begging to play Twister.

```zsh
#!/bin/zsh

# Give a Twister instruction approximately once every 5 seconds

appendages=("right hand" "right foot" "left hand" "left foot")
colors=("red" "yellow" "green" "blue" "in the air")

while true; do
    count=${#appendages[@]}
    index=$(( $RANDOM % $count + 1 ))
    appendage=${appendages[$index]}

    count=${#colors[@]}
    index=$(( $RANDOM % $count + 1 ))
    color=${colors[$index]}

    both="$appendage $color"

    echo $both
    say $both
    sleep 0.5
    say $both
    sleep 5
done
```
