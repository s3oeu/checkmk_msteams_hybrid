<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
</head>
<body>
  
  <p>
    <b>Sprache/Language:</b>
    <a href="#deutsch">Deutsch</a> |
    <a href="#english">English</a>
  </p>
  <hr>

  <!-- Deutsch -->
  <h1 id="deutsch">Teams Hybrid Notification Script für CheckMK</h1>
  <p>
    <a href="#english">→ Switch to English version</a>
  </p>
  <p>
    Dieses Python-Skript sorgt für eine zuverlässige, flexible Benachrichtigung via Microsoft Teams – speziell zugeschnitten für CheckMK.<br>
    Es kombiniert <b>sofortige Alarmierung am Tag</b> mit <b>Cache/Flush-Logik für die Nacht</b> und bietet volle Kontrolle über Ruhezeiten und Benachrichtigungsausnahmen.
  </p>
  <h2>Features</h2>
  <ul>
    <li><b>Hybrid-Mode:</b> Sofortige Benachrichtigung am Tag, Caching und manueller oder automatischer Flush nachts.</li>
    <li><b>Feiertags-/Wochenendmodus:</b> Einzelne Hosts können am Wochenende stummgeschaltet werden.</li>
    <li><b>Immer-Benachrichtigen-Liste:</b> Für kritische Hosts – unabhängig von Zeit und Modus.</li>
    <li><b>Farbige Statusanzeige und Icons</b> direkt in Teams.</li>
    <li><b>Leicht anpassbar, robust & fehlertolerant:</b> Kompatibel mit allen CheckMK-Umgebungen (auch Standalone-Flush etc.)</li>
  </ul>
  <h2>Parameterübersicht (empfohlen für CheckMK WATO)</h2>
  <table border="1" cellpadding="6" cellspacing="0">
    <tr><th>Parameter</th><th>Bedeutung</th></tr>
    <tr><td><code>1</code></td><td>Teams Webhook-URL</td></tr>
    <tr><td><code>2</code></td><td>Cache-Verzeichnis (default: /opt/omd/sites/monitoring/tmp/teams_cache)</td></tr>
    <tr><td><code>3</code></td><td>Logdatei (default: /opt/omd/sites/monitoring/var/log/teams_hybrid.log)</td></tr>
    <tr><td><code>4</code></td><td>Nachtmodus Startzeit (z.B. 22:00)</td></tr>
    <tr><td><code>5</code></td><td>Nachtmodus Ende (z.B. 07:00)</td></tr>
    <tr><td><code>6</code></td><td>Wochenende Startzeit (z.B. 10:00)</td></tr>
    <tr><td><code>7</code></td><td>Wochenende Ende (z.B. 20:00)</td></tr>
    <tr><td><code>8</code></td><td>Ignorierte Hosts am Wochenende (Komma-getrennt, z.B. <code>AVD01,AVD02</code>)</td></tr>
    <tr><td><code>9</code></td><td>Immer benachrichtigen (Komma-getrennt, z.B. <code>FW-GATE,SQL01</code>)</td></tr>
  </table>
  $1
    <li>Tagsüber werden Alarme sofort an Teams gesendet.</li>
    <li>Nachts (bzw. außerhalb der erlaubten Zeiträume) werden Probleme nur <b>gecached</b>.</li>
    <li>Ein manueller oder automatischer Flush (z.B. über CheckMK) sendet alle gespeicherten Probleme am Morgen.</li>
    <li>Hosts in der Ignorier-Liste werden am Wochenende nicht benachrichtigt, aber weiterhin im Cache abgelegt.</li>
    <li>Hosts in der Immer-Liste werden immer benachrichtigt – auch nachts/wochenends.</li>
    <li><b>Wichtig:</b> Eine OK-Benachrichtigung ("OK-Marker") wird nur dann erzeugt, wenn das ursprüngliche Problem <b>außerhalb der Sperrzeiten</b> (z.B. tagsüber) aufgetreten ist und sich während der Sperrzeit löst. Probleme, die erst in der Sperrzeit auftreten und sich dann ebenfalls während der Sperrzeit erledigen, erhalten <b>keine</b> OK-Meldung.</li>
</ol>
  <h2>Setup/Installation</h2>
  <ol>
    <li>Python 3 sowie <code>requests</code>-Modul installieren.</li>
    <li>Datei z.B. nach <code>/opt/omd/sites/monitoring/local/share/check_mk/notifications/teams_hybrid</code> kopieren.</li>
    <li>Ausführbar machen: <code>chmod +x teams_hybrid</code></li>
    <li>In CheckMK als Notification-Skript hinterlegen und Parameter im WATO setzen (siehe oben).</li>
  </ol>
  <b>Tipp:</b> Über das <b>WATO</b> können verschiedene Notification-Regeln angelegt werden (z.B. für verschiedene Teams oder unterschiedliche Ruhezeiten/Listen pro Standort).
  <h2>Flush/Manueller Aufruf</h2>
  <p>Ein Flush löscht den Cache und sendet nachts aufgelaufene Probleme nach dem Zeitplan. Typisch:</p>
  <pre><code>/opt/omd/sites/monitoring/local/share/check_mk/notifications/teams_hybrid --flush</code></pre>
  <b>Hinweis:</b> Beim manuellen Flush über die Shell sollten alle relevanten NOTIFY_PARAMETER_8 und _9 als ENV-Variable gesetzt sein, <b>oder der automatische Cache-Mechanismus des Skripts übernimmt dies</b>.<br>Meist ist ein Flush <b>über CheckMK</b> oder via Cron empfohlen.
  <h2>Beispiel-Konfiguration (CheckMK WATO)</h2>
  <pre><code>1: https://outlook.office.com/webhook/xyz/IncomingWebhook/...
2: /opt/omd/sites/monitoring/tmp/teams_cache
3: /opt/omd/sites/monitoring/var/log/teams_hybrid.log
4: 22:00
5: 07:00
6: 10:00
7: 20:00
8: AVD01,AVD02,AVD03
9: FW-GATE,SQL01
  </code></pre>
  <h2>FAQ</h2>
  <h3>Kann ich das Skript auch für andere Messenger nutzen?</h3>
  <p>Das aktuelle Layout und das Datenformat sind für Microsoft Teams optimiert. Für andere Messenger (Mattermost, Slack etc.) müsste der <code>single_card()</code>-Block angepasst werden.</p>
  <h3>Was passiert, wenn ich die Ignore-Liste (Parameter 8) ändere?</h3>
  <p>Die Änderung wirkt sich nur auf <b>neue gecachte Events</b> aus! Alte Caches können mit einem Flush gelöscht werden.</p>
  <h3>Wie stelle ich den Nacht- und Wochenendmodus ein?</h3>
  <p>Die Zeiten werden in WATO (Parameter 4–7) gesetzt. „Nacht“ gilt immer außerhalb von Tag/WE.</p>
  <h3>Kann ich mehrere Webhooks nutzen?</h3>
  <p>Ja, indem du mehrere Notification-Regeln in CheckMK anlegst und jeweils eigene Parameter verwendest.</p>
  <h3>Was muss ich tun, wenn gar nichts mehr geht?</h3>
  <p>Im Zweifel alle Caches löschen und Benachrichtigungsskript/CheckMK-Dienste neu starten. Logs prüfen!</p>
  <h2>Mitwirkende & Lizenz</h2>
  <p>Entwickelt von <b>Nils ((<a href="https://s3o.eu/">s3o.eu</a>)</b>.<br>
  <b>Lizenz:</b> MIT License – Nutzung und Anpassung ausdrücklich erlaubt!</p>

   <h2>Teams</h2>
    <!-- Screenshots -->
    <img src="https://dev.s3o.eu/dwn/github/msteams_hybrid/ok.png" alt="OK" />
    <img src="https://dev.s3o.eu/dwn/github/msteams_hybrid/warn.png" alt="Warning" />
  <hr>

  <!-- Englisch -->
  <h1 id="english">Teams Hybrid Notification Script for CheckMK</h1>
  <p>
    <a href="#deutsch">→ Zur deutschen Version</a>
  </p>
  <p>
    This Python script provides reliable, flexible Microsoft Teams notifications for CheckMK.<br>
    It combines <b>instant day alerts</b> with a <b>night cache/flush logic</b> and gives you full control over quiet hours and notification exceptions.
  </p>
  <h2>Features</h2>
  <ul>
    <li><b>Hybrid Mode:</b> Instantly notify during the day, cache and manual/automatic flush at night.</li>
    <li><b>Weekend/Holiday Mode:</b> Suppress notifications for selected hosts on weekends.</li>
    <li><b>Always-Notify List:</b> For critical hosts – notified regardless of schedule.</li>
    <li><b>Colored status and icons</b> directly in Teams.</li>
    <li><b>Easy to adapt, robust & fault-tolerant:</b> Works with all CheckMK setups (even standalone/flush etc.)</li>
  </ul>
  <h2>Parameter Overview (recommended for CheckMK WATO)</h2>
  <table border="1" cellpadding="6" cellspacing="0">
    <tr><th>Parameter</th><th>Meaning</th></tr>
    <tr><td><code>1</code></td><td>Teams webhook URL</td></tr>
    <tr><td><code>2</code></td><td>Cache directory (default: /opt/omd/sites/monitoring/tmp/teams_cache)</td></tr>
    <tr><td><code>3</code></td><td>Log file (default: /opt/omd/sites/monitoring/var/log/teams_hybrid.log)</td></tr>
    <tr><td><code>4</code></td><td>Night mode start time (e.g. 22:00)</td></tr>
    <tr><td><code>5</code></td><td>Night mode end (e.g. 07:00)</td></tr>
    <tr><td><code>6</code></td><td>Weekend mode start time (e.g. 10:00)</td></tr>
    <tr><td><code>7</code></td><td>Weekend mode end (e.g. 20:00)</td></tr>
    <tr><td><code>8</code></td><td>Ignored hosts on weekends (comma-separated, e.g. <code>AVD01,AVD02</code>)</td></tr>
    <tr><td><code>9</code></td><td>Always notify (comma-separated, e.g. <code>FW-GATE,SQL01</code>)</td></tr>
  </table>
  $1
    <li>Alerts are instantly sent to Teams during the day.</li>
    <li>At night (or outside allowed periods) problems are only <b>cached</b>.</li>
    <li>A manual or automatic flush (e.g. via CheckMK) sends all cached problems in the morning.</li>
    <li>Hosts in the ignore list are not notified on weekends, but still cached.</li>
    <li>Hosts in the always-list are always notified – day or night, weekend or not.</li>
    <li><b>Note:</b> An OK notification ("OK marker") is only created if the original problem occurred <b>outside of quiet hours</b> (e.g. daytime) and is resolved during quiet hours. Problems that first occur and are resolved <b>during</b> quiet hours will <b>not</b> generate an OK notification.</li>
</ol>
  <h2>Setup/Installation</h2>
  <ol>
    <li>Install Python 3 and the <code>requests</code> module.</li>
    <li>Copy the script to <code>/opt/omd/sites/monitoring/local/share/check_mk/notifications/teams_hybrid</code>.</li>
    <li>Make executable: <code>chmod +x teams_hybrid</code></li>
    <li>Configure as a notification script in CheckMK and set parameters in WATO (see above).</li>
  </ol>
  <b>Tip:</b> Use <b>WATO</b> to create different notification rules (for different teams or custom schedules/host lists).
  <h2>Flush/Manual Execution</h2>
  <p>A flush clears the cache and sends all collected problems according to your schedule. Example:</p>
  <pre><code>/opt/omd/sites/monitoring/local/share/check_mk/notifications/teams_hybrid --flush</code></pre>
  <b>Note:</b> For manual flush in the shell, set all NOTIFY_PARAMETER_8 and _9 as environment variables, <b>or rely on the script's internal cache logic</b>.<br>Usually, flushing <b>via CheckMK</b> or cron is recommended.
  <h2>Sample Configuration (CheckMK WATO)</h2>
  <pre><code>1: https://outlook.office.com/webhook/xyz/IncomingWebhook/...
2: /opt/omd/sites/monitoring/tmp/teams_cache
3: /opt/omd/sites/monitoring/var/log/teams_hybrid.log
4: 22:00
5: 07:00
6: 10:00
7: 20:00
8: AVD01,AVD02,AVD03
9: FW-GATE,SQL01
  </code></pre>
  <h2>FAQ</h2>
  <h3>Can I use the script for other messengers?</h3>
  <p>This layout and data format is optimized for Microsoft Teams. To support other messengers (Mattermost, Slack etc.), adapt the <code>single_card()</code> block.</p>
  <h3>What if I change the ignore-list (parameter 8)?</h3>
  <p>Changes affect only <b>newly cached events</b>! Old caches can be purged via flush.</p>
  <h3>How do I set night and weekend mode?</h3>
  <p>Times are configured via WATO (parameters 4–7). “Night” always means outside day/weekend hours.</p>
  <h3>Can I use multiple webhooks?</h3>
  <p>Yes – just create several notification rules in CheckMK with different parameters.</p>
  <h3>What if nothing works anymore?</h3>
  <p>Delete all caches and restart the notification script/CheckMK services. Check logs for details!</p>
  <h2>Contributors & License</h2>
  <p>Developed by <b>Nils (<a href="https://s3o.eu/">s3o.eu</a>)</b>.<br>
  <b>License:</b> MIT License – use and modify freely!</p>

  <h2>Teams</h2>
    <!-- Screenshots -->
    <img src="https://dev.s3o.eu/dwn/github/msteams_hybrid/ok.png" alt="OK" />
    <img src="https://dev.s3o.eu/dwn/github/msteams_hybrid/warn.png" alt="Warning" />


</body>
</html>
