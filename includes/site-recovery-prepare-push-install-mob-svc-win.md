### <a name="prepare-for-a-push-installation-on-a-windows-computer"></a>Voorbereiden voor een push-installatie op een Windows-computer

1. Zorg ervoor dat er een netwerkverbinding tussen de Windows-computer en de processerver is.
1. Maak een account dat op de processerver kan worden gebruikt voor toegang tot de computer. Het account moet beheerdersrechten, lokaal of domein hebben. Gebruik dit account alleen voor de push-installatie en agentupdates.

   > [!NOTE]
   > Als u niet een domeinaccount gebruikt, schakelt u toegangsbeheer voor externe gebruikers op de lokale computer uit. Schakel externe gebruiker Access control, onder HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System registersleutel een nieuwe DWORD toevoegen: **LocalAccountTokenFilterPolicy**. Stel de waarde voor **1**. Als u wilt deze taak achter de opdrachtprompt de volgende opdracht uitvoeren:  
   `REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1`
   >
   >
1. Selecteer in Windows Firewall op de computer die u wilt beveiligen, **toestaan een app of functie via Firewall**. Schakel **bestands- en printerdeling** en **Windows Management Instrumentation (WMI)**. Voor computers die deel uitmaken van een domein, kunt u de firewall-instellingen configureren met behulp van een groepsbeleidsobject (GPO).

   ![Firewallinstellingen](./media/site-recovery-prepare-push-install-mob-svc-win/mobility1.png)

1. Voeg het account toe dat u hebt gemaakt in CSPSConfigtool. Volg deze stappen:

    a. Meld u aan bij de configuratieserver.

    b. Open **cspsconfigtool.exe**. Het is beschikbaar als snelkoppeling op het bureaublad en in de map %ProgramData%\home\svsystems\bin.

    c. Op de **Accounts beheren** tabblad **-Account toevoegen**.

    d. Voeg het account toe dat u hebt gemaakt.

    e. Voer de referenties in die u gebruikt wanneer u replicatie voor een computer inschakelt.
