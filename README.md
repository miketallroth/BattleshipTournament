# BattleshipTournament

BattleshipTournament is a small Java training environment for new developers. Participants write an automated Battleship player, then run it against other players in matches and tournaments. The goal is to practice Java, basic strategy, and working within an existing API in a fun, low-stakes setting.

## Project Layout

The Eclipse Java project is in `BattleshipTournamentEclipseProject/BattleshipTournament`. It uses the standard `src` layout, targets JavaSE-1.8 in its Eclipse metadata, and has no external library dependencies.

The main code is in package `org.usfirst.frc.team93.training.gameplayer.battleship`:

- `GamePlayer` defines the player API and the three required overrides: `notifyReset()`, `placeShips()`, and `fireNow()`.
- `TestGamePlayer` and `TestGamePlayer2` are starter implementations. They use fixed ship placements and random shots; copy one into a uniquely named class to begin a strategy.
- `Ship`, `Ocean`, and `GameBoard` define the ship types and lengths, 10-by-10 board, placement checks, shot results, and win conditions.
- `GameBoardView` exposes only the current player's board/actions, rather than direct access to the opponent's ships.
- `Match` manages turn-taking; `Tournament` registers players, schedules round-robin matches, scores results, and writes logs.
- `CLIMain` is the command-line entry point and example tournament configuration.

## Build and Run

Use Java 8 or newer. To import in Eclipse, choose **File > Import > Existing Projects into Workspace**, select `BattleshipTournamentEclipseProject/BattleshipTournament`, then run `CLIMain` as a Java application.

Alternatively, from the repository root, compile and run from a terminal:

```sh
cd BattleshipTournamentEclipseProject/BattleshipTournament
mkdir -p bin
javac -d bin $(find src -name '*.java')
java -cp bin org.usfirst.frc.team93.training.gameplayer.battleship.CLIMain
```

The CLI currently registers `TestGamePlayer` twice and runs 10 matches between those two instances, alternating which one starts. To compare different strategies, edit the `registerPlayer(...)` calls in `src/org/usfirst/frc/team93/training/gameplayer/battleship/CLIMain.java` to register your player and an opponent. The output names are based on Java class names, so give each submitted player a distinct class name.

## Writing a Player

Create a class that extends `GamePlayer`, following either starter under `src/org/usfirst/frc/team93/training/gameplayer/battleship/players`.

- In `placeShips()`, place exactly five ships, one of each type, within the 10-by-10 grid. Ships may not overlap. Check each `placeShip(...)` result; only `Ocean.PlaceResult_t.E_OK` means the placement succeeded.
- In `fireNow()`, call `fireShot(...)` exactly once per turn. Use the returned `FireResult` (`SHOT_MISS`, `SHOT_HIT`, `SHOT_SUNK`, or `SHOT_NOT_AUTHORIZED`) and its `ship_type` to improve future shots.
- Use `notifyReset()` to clear per-match strategy state. Optionally override `notifyOpponentShot(...)` to learn from shots against your ships. `getOpponentName()` is available to a player as well.

The final player submission can be a single `.java` file, provided it is in the correct package and compiles with the game sources.

## Output and Logs

The CLI writes the tournament winner and a printable leaderboard to standard output. It does not show a graphical board or print every shot live by default. From the working directory where the Java command is run, it writes `battleship_win_loss.csv` with player records and average winning shot counts, and `battleship_match_log.csv` with one row per match.

Per-match action logs contain `ACTION` rows (whose turn and player) and `FIRE` rows (coordinates, shot result, and ship type). Open the CSV files in a text editor or spreadsheet application such as LibreOffice Calc or Excel to inspect them. `Tools/InstallBattleshipLogViewer.exe` is an optional Windows log-viewer installer; it is not needed to build or run the Java project.

The legacy logger constructs its per-match log directory using Windows-style backslashes. On Linux and macOS, those backslashes are treated as filename characters rather than directory separators, so per-match action logs may appear beside the working directory with backslashes in their names instead of inside a nested `logs` directory. The tournament summary CSV files are written to the working directory as described above. The logger path should be made platform-independent before relying on per-match log placement across operating systems.

## Training Notes

The starter strategies are deliberately basic rather than strong opponents. Improve ship placement and shot selection, keep strategy state scoped to each match, and compare players over multiple matches so the alternating first turn does not favor one side. `Tournament.buildTournamentSchedule_RoundRobin(...)` accepts a match count per player pair; use an even count for fair first-turn alternation.

