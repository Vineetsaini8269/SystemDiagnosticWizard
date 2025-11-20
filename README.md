This file is Poweshell Shell file -

<img width="1240" height="872" alt="pretestssst" src="https://github.com/user-attachments/assets/560b3cf4-8878-46b5-b8ea-e13f7a8b6eb7" />
💻 PowerShell Pretest Wizard (WPF UI)

This repository contains a self-contained PowerShell script (pretest.ps1) that launches a graphical user interface (GUI) built with Windows Presentation Foundation (WPF) to execute a series of critical system readiness checks. This tool is designed to quickly verify prerequisite environment settings before deploying or running a complex application suite.

✨ Features

The Pretest Wizard provides a visual dashboard to run and track the status of six essential system checks, executed non-blockingly using PowerShell Jobs.

Index

Test Name

Description

1

Check Disk (D:)

Verifies the existence of the D: drive and reports the available free space.

2

Check Internet

Attempts to reach http://www.google.com to confirm external network connectivity.

3

Speed Test (Local Write)

Measures local disk write performance by creating a temporary 10MB file and calculating the speed in MB/s.

4

Check Node.js

Executes node -v to confirm Node.js is installed and accessible in the system path.

5

Check Java

Executes java -version to confirm the Java runtime is installed and accessible in the system path.

6

Check API Endpoint

Attempts an Invoke-WebRequest to a public endpoint (like https://www.youtube.com) to check outbound HTTP requests and timeout handling.

🚀 Usage

Prerequisites: You must be running PowerShell 5.1 or later (PowerShell Core / 7+ is also compatible, though the script was originally written for Windows PowerShell).

Execution: Run the script directly from your PowerShell console:

.\pretest.ps1


Interaction:

Click any individual Run X: button to execute a single test in isolation.

Click RUN ALL TESTS to execute all checks sequentially.

Use Reset All Status to clear the dashboard for a new run.

⚙️ Technical Overview

Architecture

The application is built using a combination of PowerShell for logic and XAML for the User Interface.

UI: The graphical interface is defined in a single, embedded XAML string, loaded via [Windows.Markup.XamlReader]::Load().

Non-Blocking Execution: All tests (Test1_Disk_Logic through Test6_API_Logic) are executed as PowerShell Jobs (Start-Job). This is crucial because it prevents the main WPF UI thread from freezing while tests (which involve network requests or disk I/O) are running.

Key Design Note: Job Scoping and UI Updates

A core technical challenge in mixing PowerShell Jobs and WPF is managing variable scope and thread access.

The UI control objects ($SummaryText, $Detail1, etc.) exist only on the main thread of the parent PowerShell process. When a PowerShell Job starts, it is a completely separate process that loses direct access to those UI objects.

The solution implemented is:

Decoupled Logic: The test logic blocks ($TestX_Logic) only use Write-Output to send status messages (e.g., @{ Action = 'Detail'; ... }) back to the parent process.

Job Polling: The Run-Test-Job function continuously polls the job using Receive-Job -Keep.

Thread Safety: Upon receiving a status message, the parent process uses the dedicated UI Dispatcher ($UI.Invoke) to safely update the UI controls.

Variable Bridging: For wrapper functions called inside the non-blocking job (Run-Single-Test-Job), global functions and UI references are explicitly made available using the $using: scope modifier, ensuring the functions can correctly call the thread-safe UiInvoke methods in the main process. This resolves the common The property 'Text' cannot be found on this object error.

This structure guarantees a responsive, professional-grade desktop application experience while leveraging the robust scripting capabilities of PowerShell.
