# Introduction

In this lab, students will explore where Linux passwords are stored and learn about tools and techniques for cracking them. Protecting passwords, especially the root password, is essential for system security.

Linux user accounts are listed in the passwd file, found in the /etc directory, while encrypted password hashes are stored in the more secure shadow file. Additionally, SSH connections and security-related events are logged in the auth.log file. Students will also use John the Ripper, a powerful password-cracking tool, to understand dictionary and brute-force attacks.

