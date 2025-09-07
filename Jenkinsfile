pipeline {
  agent any
  options { ansiColor('xterm'); timestamps() }

  // ⚡ Planification : tous les jours à 08h et 18h (heure Paris)
  triggers {
    cron('''TZ=Europe/Paris
0 8 * * 1-7
0 18 * * 1-7
''')
  }

