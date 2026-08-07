<script>
  import { onMount } from 'svelte';

  const STORAGE_KEY = 'chess-tournament-manager-v1';

  let players = [];
  let tournaments = [];
  let activeTab = 'dashboard';

  let playerName = '';
  let playerEmail = '';
  let playerRating = '';

  let tournamentName = '';
  let tournamentDate = '';

  let selectedTournamentId = '';
  let editingPlayerId = null;
  let editingTournamentId = null;
  let message = '';
  let messageType = 'success';

  function save() {
    localStorage.setItem(STORAGE_KEY, JSON.stringify({ players, tournaments }));
  }

  function notify(text, type = 'success') {
    message = text;
    messageType = type;
    setTimeout(() => (message = ''), 2800);
  }

  function uid(prefix) {
    return `${prefix}-${Date.now()}-${Math.random().toString(36).slice(2, 8)}`;
  }

  function addOrUpdatePlayer() {
    const name = playerName.trim();
    const email = playerEmail.trim();
    const rating = Number(playerRating);

    if (!name || !email || !rating) {
      notify('Enter player name, email and rating.', 'error');
      return;
    }

    if (editingPlayerId) {
      players = players.map((p) =>
        p.id === editingPlayerId ? { ...p, name, email, rating } : p
      );
      notify('Player updated successfully.');
    } else {
      players = [...players, { id: uid('p'), name, email, rating }];
      notify('Player created successfully.');
    }

    resetPlayerForm();
    save();
  }

  function editPlayer(player) {
    editingPlayerId = player.id;
    playerName = player.name;
    playerEmail = player.email;
    playerRating = player.rating;
    activeTab = 'players';
  }

  function deletePlayer(id) {
    const player = players.find((p) => p.id === id);
    if (!player) return;
    if (!confirm(`Delete ${player.name}?`)) return;

    players = players.filter((p) => p.id !== id);
    tournaments = tournaments.map((t) => ({
      ...t,
      playerIds: t.playerIds.filter((playerId) => playerId !== id)
    }));
    save();
    notify('Player deleted.');
  }

  function resetPlayerForm() {
    editingPlayerId = null;
    playerName = '';
    playerEmail = '';
    playerRating = '';
  }

  function addOrUpdateTournament() {
    const name = tournamentName.trim();
    if (!name || !tournamentDate) {
      notify('Enter tournament name and date.', 'error');
      return;
    }

    if (editingTournamentId) {
      tournaments = tournaments.map((t) =>
        t.id === editingTournamentId ? { ...t, name, date: tournamentDate } : t
      );
      notify('Tournament updated successfully.');
    } else {
      const id = uid('t');
      tournaments = [
        ...tournaments,
        { id, name, date: tournamentDate, playerIds: [], matches: [], completed: false }
      ];
      selectedTournamentId = id;
      notify('Tournament created successfully.');
    }

    resetTournamentForm();
    save();
  }

  function editTournament(tournament) {
    editingTournamentId = tournament.id;
    tournamentName = tournament.name;
    tournamentDate = tournament.date;
    activeTab = 'tournaments';
  }

  function deleteTournament(id) {
    const tournament = tournaments.find((t) => t.id === id);
    if (!tournament) return;
    if (!confirm(`Delete ${tournament.name}?`)) return;

    tournaments = tournaments.filter((t) => t.id !== id);
    if (selectedTournamentId === id) selectedTournamentId = tournaments[0]?.id ?? '';
    save();
    notify('Tournament deleted.');
  }

  function resetTournamentForm() {
    editingTournamentId = null;
    tournamentName = '';
    tournamentDate = '';
  }

  function getTournament(id) {
    return tournaments.find((t) => t.id === id);
  }

  function addPlayerToTournament(playerId) {
    const tournament = getTournament(selectedTournamentId);
    if (!tournament) return;

    if (tournament.playerIds.includes(playerId)) {
      notify('Player is already in this tournament.', 'error');
      return;
    }

    tournaments = tournaments.map((t) =>
      t.id === tournament.id
        ? { ...t, playerIds: [...t.playerIds, playerId], completed: false }
        : t
    );
    save();
    notify('Player added to tournament.');
  }

  function removePlayerFromTournament(playerId) {
    const tournament = getTournament(selectedTournamentId);
    if (!tournament || tournament.matches.length) {
      notify('Players cannot be removed after matches are generated.', 'error');
      return;
    }

    tournaments = tournaments.map((t) =>
      t.id === tournament.id
        ? { ...t, playerIds: t.playerIds.filter((id) => id !== playerId) }
        : t
    );
    save();
  }

  function shuffle(list) {
    const copy = [...list];
    for (let i = copy.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [copy[i], copy[j]] = [copy[j], copy[i]];
    }
    return copy;
  }

  function generateMatches() {
    const tournament = getTournament(selectedTournamentId);
    if (!tournament) {
      notify('Select a tournament first.', 'error');
      return;
    }

    if (tournament.playerIds.length < 2) {
      notify('Add at least 2 players before generating matches.', 'error');
      return;
    }

    if (tournament.matches.length && !confirm('Regenerate all matches and results?')) return;

    const participantIds = shuffle(tournament.playerIds);
    const matches = [];

    for (let i = 0; i < participantIds.length; i += 2) {
      const playerA = participantIds[i];
      const playerB = participantIds[i + 1];

      if (!playerB) {
        matches.push({
          id: uid('m'),
          playerA,
          playerB: null,
          winner: playerA,
          status: 'BYE'
        });
      } else {
        const winner = Math.random() < 0.5 ? playerA : playerB;
        matches.push({
          id: uid('m'),
          playerA,
          playerB,
          winner,
          status: 'COMPLETED'
        });
      }
    }

    tournaments = tournaments.map((t) =>
      t.id === tournament.id ? { ...t, matches, completed: true } : t
    );
    save();
    notify('Random matches and winners generated.');
  }

  function playerNameById(id) {
    return players.find((p) => p.id === id)?.name ?? 'Unknown player';
  }

  function standings(tournament) {
    const scores = new Map();
    tournament.playerIds.forEach((id) => scores.set(id, 0));

    tournament.matches.forEach((match) => {
      if (match.winner) scores.set(match.winner, (scores.get(match.winner) ?? 0) + 1);
    });

    return [...scores.entries()]
      .map(([id, wins]) => ({
        id,
        name: playerNameById(id),
        wins,
        rating: players.find((p) => p.id === id)?.rating ?? 0
      }))
      .sort((a, b) => b.wins - a.wins || b.rating - a.rating || a.name.localeCompare(b.name));
  }

  $: selectedTournament = getTournament(selectedTournamentId);
  $: selectedPlayerIds = selectedTournament?.playerIds ?? [];
  $: availablePlayers = players.filter((p) => !selectedPlayerIds.includes(p.id));
  $: topStandings = selectedTournament ? standings(selectedTournament).slice(0, 3) : [];

  onMount(() => {
    try {
      const saved = JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}');
      players = Array.isArray(saved.players) ? saved.players : [];
      tournaments = Array.isArray(saved.tournaments) ? saved.tournaments : [];
      selectedTournamentId = tournaments[0]?.id ?? '';
    } catch {
      players = [];
      tournaments = [];
    }
  });
</script>

<svelte:head>
  <title>Chess Tournament Manager</title>
</svelte:head>

<div class="app-shell">
  <header class="topbar">
    <div>
      <p class="eyebrow">TOURNAMENT CONTROL</p>
      <h1>♟ Chess Tournament Manager</h1>
      <p class="subtitle">Players, tournaments, random matches and final rankings.</p>
    </div>
    <div class="status-pill"><span></span> Local data saved</div>
  </header>

  <nav class="tabs" aria-label="Main navigation">
    {#each [
      ['dashboard', 'Dashboard'],
      ['players', 'Players'],
      ['tournaments', 'Tournaments'],
      ['matches', 'Matches'],
      ['rankings', 'Rankings']
    ] as tab}
      <button class:active={activeTab === tab[0]} on:click={() => (activeTab = tab[0])}>
        {tab[1]}
      </button>
    {/each}
  </nav>

  {#if message}
    <div class:error={messageType === 'error'} class="toast">{message}</div>
  {/if}

  {#if activeTab === 'dashboard'}
    <section class="hero-grid">
      <div class="hero-card">
        <p class="eyebrow">ASSIGNMENT READY</p>
        <h2>Manage a complete chess tournament in one place.</h2>
        <p>Create players and tournaments, add participants, generate random pairings, select random winners and view the top three.</p>
        <button class="primary" on:click={() => (activeTab = 'players')}>Start with Players →</button>
      </div>

      <div class="stats-grid">
        <div class="stat-card"><span>Players</span><strong>{players.length}</strong></div>
        <div class="stat-card"><span>Tournaments</span><strong>{tournaments.length}</strong></div>
        <div class="stat-card"><span>Matches</span><strong>{tournaments.reduce((sum, t) => sum + t.matches.length, 0)}</strong></div>
        <div class="stat-card"><span>Completed</span><strong>{tournaments.filter((t) => t.completed).length}</strong></div>
      </div>
    </section>

    <section class="panel">
      <div class="section-heading">
        <div>
          <p class="eyebrow">QUICK FLOW</p>
          <h3>How the system works</h3>
        </div>
      </div>
      <div class="flow">
        <div><b>01</b><span>Create players</span></div>
        <div><b>02</b><span>Create a tournament</span></div>
        <div><b>03</b><span>Add participants</span></div>
        <div><b>04</b><span>Generate random matches</span></div>
        <div><b>05</b><span>Review top 3 ranking</span></div>
      </div>
    </section>
  {:else if activeTab === 'players'}
    <section class="page-grid">
      <div class="panel form-panel">
        <p class="eyebrow">PLAYER MANAGEMENT</p>
        <h2>{editingPlayerId ? 'Edit player' : 'Add player'}</h2>
        <label>Name<input bind:value={playerName} placeholder="e.g. Arjun Kumar" /></label>
        <label>Email<input type="email" bind:value={playerEmail} placeholder="player@example.com" /></label>
        <label>Rating<input type="number" min="1" bind:value={playerRating} placeholder="1200" /></label>
        <div class="button-row">
          <button class="primary" on:click={addOrUpdatePlayer}>{editingPlayerId ? 'Update Player' : 'Create Player'}</button>
          {#if editingPlayerId}<button class="secondary" on:click={resetPlayerForm}>Cancel</button>{/if}
        </div>
      </div>

      <div class="panel">
        <div class="section-heading"><div><p class="eyebrow">CRUD</p><h2>Players ({players.length})</h2></div></div>
        {#if players.length === 0}
          <div class="empty">No players yet. Create your first player.</div>
        {:else}
          <div class="table-wrap">
            <table>
              <thead><tr><th>Player</th><th>Email</th><th>Rating</th><th>Actions</th></tr></thead>
              <tbody>
                {#each players as player}
                  <tr>
                    <td><strong>{player.name}</strong></td><td>{player.email}</td><td>{player.rating}</td>
                    <td class="actions"><button on:click={() => editPlayer(player)}>Edit</button><button class="danger" on:click={() => deletePlayer(player.id)}>Delete</button></td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        {/if}
      </div>
    </section>
  {:else if activeTab === 'tournaments'}
    <section class="page-grid">
      <div class="panel form-panel">
        <p class="eyebrow">TOURNAMENT MANAGEMENT</p>
        <h2>{editingTournamentId ? 'Edit tournament' : 'Create tournament'}</h2>
        <label>Name<input bind:value={tournamentName} placeholder="e.g. Summer Chess Cup" /></label>
        <label>Date<input type="date" bind:value={tournamentDate} /></label>
        <div class="button-row">
          <button class="primary" on:click={addOrUpdateTournament}>{editingTournamentId ? 'Update Tournament' : 'Create Tournament'}</button>
          {#if editingTournamentId}<button class="secondary" on:click={resetTournamentForm}>Cancel</button>{/if}
        </div>
      </div>

      <div class="panel">
        <div class="section-heading"><div><p class="eyebrow">CRUD</p><h2>Tournaments ({tournaments.length})</h2></div></div>
        {#if tournaments.length === 0}
          <div class="empty">No tournaments yet. Create one to continue.</div>
        {:else}
          <div class="cards-list">
            {#each tournaments as tournament}
              <article class:selected={selectedTournamentId === tournament.id} class="item-card">
                <div><h3>{tournament.name}</h3><p>{tournament.date} · {tournament.playerIds.length} players · {tournament.matches.length} matches</p></div>
                <div class="actions"><button on:click={() => { selectedTournamentId = tournament.id; activeTab = 'matches'; }}>Manage</button><button on:click={() => editTournament(tournament)}>Edit</button><button class="danger" on:click={() => deleteTournament(tournament.id)}>Delete</button></div>
              </article>
            {/each}
          </div>
        {/if}
      </div>
    </section>
  {:else if activeTab === 'matches'}
    <section class="panel">
      <div class="section-heading">
        <div><p class="eyebrow">MATCH SYSTEM</p><h2>Random pairings & winners</h2></div>
        <select bind:value={selectedTournamentId}>
          <option value="">Select tournament</option>
          {#each tournaments as tournament}<option value={tournament.id}>{tournament.name}</option>{/each}
        </select>
      </div>

      {#if selectedTournament}
        <div class="match-toolbar">
          <div><strong>{selectedTournament.name}</strong><span>{selectedTournament.playerIds.length} participants</span></div>
          <button class="primary" on:click={generateMatches}>🎲 Generate Random Matches</button>
        </div>

        {#if availablePlayers.length}
          <div class="add-box">
            <strong>Add players to this tournament</strong>
            <div class="player-picker">
              {#each availablePlayers as player}
                <button on:click={() => addPlayerToTournament(player.id)}>+ {player.name}</button>
              {/each}
            </div>
          </div>
        {/if}

        {#if selectedPlayerIds.length}
          <div class="participant-list">
            {#each selectedPlayerIds as id}
              <span>{playerNameById(id)} {#if !selectedTournament.matches.length}<button aria-label="Remove player" on:click={() => removePlayerFromTournament(id)}>×</button>{/if}</span>
            {/each}
          </div>
        {/if}

        {#if selectedTournament.matches.length === 0}
          <div class="empty">No matches generated. Add at least two players, then click Generate Random Matches.</div>
        {:else}
          <div class="matches-grid">
            {#each selectedTournament.matches as match, i}
              <article class="match-card">
                <span class="round">MATCH {i + 1}</span>
                <div class="versus">
                  <div class:winner={match.winner === match.playerA}><strong>{playerNameById(match.playerA)}</strong><small>{match.winner === match.playerA ? 'WINNER' : 'PLAYER'}</small></div>
                  <b>VS</b>
                  <div class:bye={match.status === 'BYE'} class:winner={match.winner === match.playerB}><strong>{match.playerB ? playerNameById(match.playerB) : 'BYE'}</strong><small>{match.status === 'BYE' ? 'BYE' : match.winner === match.playerB ? 'WINNER' : 'PLAYER'}</small></div>
                </div>
              </article>
            {/each}
          </div>
        {/if}
      {:else}
        <div class="empty">Create a tournament first.</div>
      {/if}
    </section>
  {:else if activeTab === 'rankings'}
    <section class="panel">
      <div class="section-heading">
        <div><p class="eyebrow">FINAL RANKINGS</p><h2>Top 3 players</h2></div>
        <select bind:value={selectedTournamentId}>
          <option value="">Select tournament</option>
          {#each tournaments as tournament}<option value={tournament.id}>{tournament.name}</option>{/each}
        </select>
      </div>

      {#if selectedTournament && selectedTournament.matches.length}
        <div class="podium">
          {#each topStandings as row, index}
            <article class:gold={index === 0} class:silver={index === 1} class:bronze={index === 2} class="rank-card">
              <div class="rank-number">{index + 1}</div>
              <div><span class="medal">{['🥇','🥈','🥉'][index]}</span><h3>{row.name}</h3><p>{row.wins} win{row.wins === 1 ? '' : 's'} · Rating {row.rating}</p></div>
            </article>
          {/each}
        </div>
        {#if topStandings.length < 3}<p class="hint">Only {topStandings.length} ranked player(s) are available. Add more participants for a full top 3.</p>{/if}
      {:else}
        <div class="empty">Generate tournament matches first to calculate rankings.</div>
      {/if}
    </section>
  {/if}

  <footer>Chess Tournament Management System · Svelte + JavaScript · Data persists in browser localStorage</footer>
</div>
