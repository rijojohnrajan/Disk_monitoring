# Cloud VM Disk Utilization Monitoring — AWS (Ansible + CloudWatch)

A secure, multi-account, scalable solution to monitor disk utilization across **all EC2 VMs** in an AWS Organization, detect low disk space early, and minimize downtime risk.

It leverages the **existing stack (Ansible)** for discovery + enrollment and adds only the **cloud-native services that clearly earn their place** (SSM, CloudWatch, OAM, SNS).

---

## 1. High-Level Architecture



![Architecture](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMTgwIiBoZWlnaHQ9Ijc2MCIgdmlld0JveD0iMCAwIDExODAgNzYwIiBmb250LWZhbWlseT0iU2Vnb2UgVUksIEFyaWFsLCBzYW5zLXNlcmlmIj4KICA8ZGVmcz4KICAgIDxtYXJrZXIgaWQ9ImFycm93IiBtYXJrZXJXaWR0aD0iMTAiIG1hcmtlckhlaWdodD0iMTAiIHJlZlg9IjgiIHJlZlk9IjMiIG9yaWVudD0iYXV0byIgbWFya2VyVW5pdHM9InN0cm9rZVdpZHRoIj4KICAgICAgPHBhdGggZD0iTTAsMCBMOCwzIEwwLDYgWiIgZmlsbD0iIzU1NSIvPgogICAgPC9tYXJrZXI+CiAgICA8c3R5bGU+CiAgICAgIC50aXRsZXtmb250LXNpemU6MjJweDtmb250LXdlaWdodDo3MDA7ZmlsbDojMTYxOTFmfQogICAgICAuc3Vie2ZvbnQtc2l6ZToxMnB4O2ZpbGw6IzU1Nn0KICAgICAgLmFjY3R7Zm9udC1zaXplOjEzcHg7Zm9udC13ZWlnaHQ6NzAwO2ZpbGw6IzBkM2I2Nn0KICAgICAgLmxibHtmb250LXNpemU6MTJweDtmaWxsOiMxNjE5MWZ9CiAgICAgIC5zbWFsbHtmb250LXNpemU6MTFweDtmaWxsOiM0NDV9CiAgICAgIC5mbG93e3N0cm9rZTojNTU1O3N0cm9rZS13aWR0aDoxLjY7ZmlsbDpub25lO21hcmtlci1lbmQ6dXJsKCNhcnJvdyl9CiAgICAgIC5mbG93ZHtzdHJva2U6I2UwN2IwMDtzdHJva2Utd2lkdGg6MS42O2ZpbGw6bm9uZTtzdHJva2UtZGFzaGFycmF5OjUgNDttYXJrZXItZW5kOnVybCgjYXJyb3cpfQogICAgPC9zdHlsZT4KICA8L2RlZnM+CgogIDx0ZXh0IHg9IjMwIiB5PSIzNCIgY2xhc3M9InRpdGxlIj5BV1MgTXVsdGktQWNjb3VudCBEaXNrIFV0aWxpemF0aW9uIE1vbml0b3Jpbmcg4oCUIEFuc2libGUgKyBDbG91ZFdhdGNoPC90ZXh0PgogIDx0ZXh0IHg9IjMwIiB5PSI1NCIgY2xhc3M9InN1YiI+S2V5bGVzcyBhY2Nlc3MgdmlhIFNTTSDCtyBjb250aW51b3VzIENsb3VkV2F0Y2ggQWdlbnQgbWV0cmljcyDCtyBjcm9zcy1hY2NvdW50IGFnZ3JlZ2F0aW9uIHZpYSBPQU0gwrcgU05TIGFsZXJ0aW5nIMK3IFN0YWNrU2V0IHNjYWxpbmc8L3RleHQ+CgogIDxyZWN0IHg9IjMwIiB5PSI4MCIgd2lkdGg9IjQ3MCIgaGVpZ2h0PSIzMDAiIHJ4PSIxMCIgZmlsbD0iI2VhZjJmYiIgc3Ryb2tlPSIjMGQzYjY2IiBzdHJva2Utd2lkdGg9IjEuNSIvPgogIDx0ZXh0IHg9IjQ4IiB5PSIxMDQiIGNsYXNzPSJhY2N0Ij5Nb25pdG9yaW5nIC8gVG9vbGluZyBBY2NvdW50ICgxMTExMjIyMjMzMzMpPC90ZXh0PgoKICA8cmVjdCB4PSI1MiIgeT0iMTIwIiB3aWR0aD0iMTgwIiBoZWlnaHQ9IjcwIiByeD0iOCIgZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjM2I2ZWE1Ii8+CiAgPHRleHQgeD0iNjIiIHk9IjE0MiIgY2xhc3M9ImxibCIgZm9udC13ZWlnaHQ9IjcwMCI+QW5zaWJsZSBDb250cm9sIE5vZGU8L3RleHQ+CiAgPHRleHQgeD0iNjIiIHk9IjE2MCIgY2xhc3M9InNtYWxsIj5hd3NfZWMyIGR5bmFtaWMgaW52ZW50b3J5PC90ZXh0PgogIDx0ZXh0IHg9IjYyIiB5PSIxNzYiIGNsYXNzPSJzbWFsbCI+YXdzX3NzbSBjb25uZWN0aW9uIChubyBTU0gpPC90ZXh0PgoKICA8cmVjdCB4PSIyNTAiIHk9IjEyMCIgd2lkdGg9IjIzMCIgaGVpZ2h0PSI3MCIgcng9IjgiIGZpbGw9IiNmZmZmZmYiIHN0cm9rZT0iIzNiNmVhNSIvPgogIDx0ZXh0IHg9IjI2MCIgeT0iMTQyIiBjbGFzcz0ibGJsIiBmb250LXdlaWdodD0iNzAwIj5DbG91ZFdhdGNoIChjZW50cmFsKTwvdGV4dD4KICA8dGV4dCB4PSIyNjAiIHk9IjE2MCIgY2xhc3M9InNtYWxsIj5PQU0gU2luayDCtyBjcm9zcy1hY2NvdW50IGRhc2hib2FyZHM8L3RleHQ+CiAgPHRleHQgeD0iMjYwIiB5PSIxNzYiIGNsYXNzPSJzbWFsbCI+TWV0cmljcyBJbnNpZ2h0czogZnVsbGVzdCB2b2x1bWVzIHRvcC1OPC90ZXh0PgoKICA8cmVjdCB4PSI1MiIgeT0iMjEwIiB3aWR0aD0iMTgwIiBoZWlnaHQ9IjY2IiByeD0iOCIgZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjM2I2ZWE1Ii8+CiAgPHRleHQgeD0iNjIiIHk9IjIzMiIgY2xhc3M9ImxibCIgZm9udC13ZWlnaHQ9IjcwMCI+Q2xvdWRXYXRjaCBBbGFybXM8L3RleHQ+CiAgPHRleHQgeD0iNjIiIHk9IjI1MCIgY2xhc3M9InNtYWxsIj5kaXNrX3VzZWRfcGVyY2VudCAmZ3Q7PSA4MCU8L3RleHQ+CiAgPHRleHQgeD0iNjIiIHk9IjI2NiIgY2xhc3M9InNtYWxsIj5UcmVhdE1pc3NpbmdEYXRhOiBicmVhY2hpbmc8L3RleHQ+CgogIDxyZWN0IHg9IjI1MCIgeT0iMjEwIiB3aWR0aD0iMjMwIiBoZWlnaHQ9IjY2IiByeD0iOCIgZmlsbD0iI2ZmZjRlNSIgc3Ryb2tlPSIjZTA3YjAwIi8+CiAgPHRleHQgeD0iMjYwIiB5PSIyMzIiIGNsYXNzPSJsYmwiIGZvbnQtd2VpZ2h0PSI3MDAiPlNOUyBUb3BpYyDihpIgbm90aWZpY2F0aW9uczwvdGV4dD4KICA8dGV4dCB4PSIyNjAiIHk9IjI1MCIgY2xhc3M9InNtYWxsIj5lbWFpbCDCtyBTbGFjayDCtyBQYWdlckR1dHkgwrcgdGlja2V0aW5nPC90ZXh0PgoKICA8cmVjdCB4PSI1MiIgeT0iMjk2IiB3aWR0aD0iNDI4IiBoZWlnaHQ9IjY2IiByeD0iOCIgZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjM2I2ZWE1Ii8+CiAgPHRleHQgeD0iNjIiIHk9IjMxOCIgY2xhc3M9ImxibCIgZm9udC13ZWlnaHQ9IjcwMCI+R292ZXJuYW5jZSAoT3JnYW5pemF0aW9ucyk8L3RleHQ+CiAgPHRleHQgeD0iNjIiIHk9IjMzNiIgY2xhc3M9InNtYWxsIj5DbG91ZEZvcm1hdGlvbiBTdGFja1NldHMg4oaSIElBTSByb2xlcywgaW5zdGFuY2UgcHJvZmlsZXMsIE9BTSBsaW5rczwvdGV4dD4KICA8dGV4dCB4PSI2MiIgeT0iMzUyIiBjbGFzcz0ic21hbGwiPlNTTSBTdGF0ZSBNYW5hZ2VyIGtlZXBzIENXIEFnZW50IGluc3RhbGxlZCAoc2VsZi1oZWFsaW5nKTwvdGV4dD4KCiAgPGc+CiAgICA8cmVjdCB4PSI2NDAiIHk9IjgwIiB3aWR0aD0iNTEwIiBoZWlnaHQ9IjE1MCIgcng9IjEwIiBmaWxsPSIjZjJmOGYyIiBzdHJva2U9IiMyZTdkMzIiIHN0cm9rZS13aWR0aD0iMS41Ii8+CiAgICA8dGV4dCB4PSI2NTgiIHk9IjEwNCIgY2xhc3M9ImFjY3QiIGZpbGw9IiMxYjVlMjAiPk1lbWJlciBBY2NvdW50IEEg4oCUIFZQQyAocHJpdmF0ZSBzdWJuZXRzKTwvdGV4dD4KICAgIDxyZWN0IHg9IjY2MCIgeT0iMTE4IiB3aWR0aD0iMTQwIiBoZWlnaHQ9Ijk2IiByeD0iOCIgZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjMmU3ZDMyIi8+CiAgICA8dGV4dCB4PSI2NzAiIHk9IjE0MCIgY2xhc3M9ImxibCIgZm9udC13ZWlnaHQ9IjcwMCI+RUMyIFZNPC90ZXh0PgogICAgPHRleHQgeD0iNjcwIiB5PSIxNTgiIGNsYXNzPSJzbWFsbCI+U1NNIEFnZW50PC90ZXh0PgogICAgPHRleHQgeD0iNjcwIiB5PSIxNzQiIGNsYXNzPSJzbWFsbCI+Q2xvdWRXYXRjaCBBZ2VudDwvdGV4dD4KICAgIDx0ZXh0IHg9IjY3MCIgeT0iMTkwIiBjbGFzcz0ic21hbGwiPkluc3RhbmNlIHByb2ZpbGU8L3RleHQ+CiAgICA8dGV4dCB4PSI2NzAiIHk9IjIwNiIgY2xhc3M9InNtYWxsIj50YWcgTW9uaXRvcmluZz1FbmFibGVkPC90ZXh0PgogICAgPHJlY3QgeD0iODEyIiB5PSIxMTgiIHdpZHRoPSIxNDAiIGhlaWdodD0iOTYiIHJ4PSI4IiBmaWxsPSIjZmZmZmZmIiBzdHJva2U9IiMyZTdkMzIiLz4KICAgIDx0ZXh0IHg9IjgyMiIgeT0iMTQwIiBjbGFzcz0ibGJsIiBmb250LXdlaWdodD0iNzAwIj5FQzIgVk08L3RleHQ+CiAgICA8dGV4dCB4PSI4MjIiIHk9IjE1OCIgY2xhc3M9InNtYWxsIj5TU00gQWdlbnQ8L3RleHQ+CiAgICA8dGV4dCB4PSI4MjIiIHk9IjE3NCIgY2xhc3M9InNtYWxsIj5DbG91ZFdhdGNoIEFnZW50PC90ZXh0PgogICAgPHRleHQgeD0iODIyIiB5PSIxOTAiIGNsYXNzPSJzbWFsbCI+ZGlza191c2VkX3BlcmNlbnQgNjBzPC90ZXh0PgogICAgPHJlY3QgeD0iOTY0IiB5PSIxMTgiIHdpZHRoPSIxNjYiIGhlaWdodD0iOTYiIHJ4PSI4IiBmaWxsPSIjZmZmZmZmIiBzdHJva2U9IiMyZTdkMzIiLz4KICAgIDx0ZXh0IHg9Ijk3NCIgeT0iMTQwIiBjbGFzcz0ibGJsIiBmb250LXdlaWdodD0iNzAwIj5BY2NvdW50IHNlcnZpY2VzPC90ZXh0PgogICAgPHRleHQgeD0iOTc0IiB5PSIxNTgiIGNsYXNzPSJzbWFsbCI+eEFjY3QgSUFNIHJvbGUgKEFzc3VtZVJvbGUpPC90ZXh0PgogICAgPHRleHQgeD0iOTc0IiB5PSIxNzQiIGNsYXNzPSJzbWFsbCI+Q2xvdWRXYXRjaCAobG9jYWwgbWV0cmljcyk8L3RleHQ+CiAgICA8dGV4dCB4PSI5NzQiIHk9IjE5MCIgY2xhc3M9InNtYWxsIj5PQU0gTGluayDihpIgU2luazwvdGV4dD4KICAgIDx0ZXh0IHg9Ijk3NCIgeT0iMjA2IiBjbGFzcz0ic21hbGwiPlNTTSBlbmRwb2ludHM8L3RleHQ+CiAgPC9nPgoKICA8Zz4KICAgIDxyZWN0IHg9IjY0MCIgeT0iMjUwIiB3aWR0aD0iNTEwIiBoZWlnaHQ9IjEyMCIgcng9IjEwIiBmaWxsPSIjZjJmOGYyIiBzdHJva2U9IiMyZTdkMzIiIHN0cm9rZS13aWR0aD0iMS41Ii8+CiAgICA8dGV4dCB4PSI2NTgiIHk9IjI3NCIgY2xhc3M9ImFjY3QiIGZpbGw9IiMxYjVlMjAiPk1lbWJlciBBY2NvdW50IEIgLyBDIOKApiAoYXV0by1lbnJvbGxlZCBvbiBhY2NvdW50IGNyZWF0aW9uKTwvdGV4dD4KICAgIDxyZWN0IHg9IjY2MCIgeT0iMjg4IiB3aWR0aD0iMTQwIiBoZWlnaHQ9IjY2IiByeD0iOCIgZmlsbD0iI2ZmZmZmZiIgc3Ryb2tlPSIjMmU3ZDMyIi8+CiAgICA8dGV4dCB4PSI2NzAiIHk9IjMxMCIgY2xhc3M9ImxibCIgZm9udC13ZWlnaHQ9IjcwMCI+RUMyIFZNczwvdGV4dD4KICAgIDx0ZXh0IHg9IjY3MCIgeT0iMzI4IiBjbGFzcz0ic21hbGwiPlNTTSArIENXIEFnZW50PC90ZXh0PgogICAgPHRleHQgeD0iNjcwIiB5PSIzNDQiIGNsYXNzPSJzbWFsbCI+T0FNIExpbmsg4oaSIFNpbms8L3RleHQ+CiAgICA8cmVjdCB4PSI4MTIiIHk9IjI4OCIgd2lkdGg9IjMxOCIgaGVpZ2h0PSI2NiIgcng9IjgiIGZpbGw9IiNlZWY0ZmYiIHN0cm9rZT0iIzNiNmVhNSIgc3Ryb2tlLWRhc2hhcnJheT0iNCAzIi8+CiAgICA8dGV4dCB4PSI4MjIiIHk9IjMxMCIgY2xhc3M9InNtYWxsIj5TYW1lIFN0YWNrU2V0LWRlcGxveWVkIHJvbGVzLCBwcm9maWxlcywgYW5kIE9BTSBsaW5rLjwvdGV4dD4KICAgIDx0ZXh0IHg9IjgyMiIgeT0iMzI4IiBjbGFzcz0ic21hbGwiPk5ldyB0YWdnZWQgVk0g4oaSIGRpc2NvdmVyZWQgYnkgYXdzX2VjMiBpbnZlbnRvcnkg4oaSIGVucm9sbGVkPC90ZXh0PgogICAgPHRleHQgeD0iODIyIiB5PSIzNDQiIGNsYXNzPSJzbWFsbCI+YnkgY2xvdWR3YXRjaF9hZ2VudCByb2xlIOKGkiBtZXRyaWNzIGZsb3cgdG8gY2VudHJhbCBzaW5rLjwvdGV4dD4KICA8L2c+CgogIDxwYXRoIGNsYXNzPSJmbG93IiBkPSJNMjMyIDE1MCBDIDQ2MCAxMzAsIDUyMCAxNTAsIDY2MCAxNTgiLz4KICA8dGV4dCB4PSI0NzAiIHk9IjEyNiIgY2xhc3M9InNtYWxsIj5Bc3N1bWVSb2xlICsgYXdzX3NzbSAoa2V5bGVzcywgcG9ydGxlc3MpPC90ZXh0PgogIDxwYXRoIGNsYXNzPSJmbG93IiBkPSJNOTUyIDE2MCBDIDk5MCAxNjAsIDk5NSAxNTAsIDEwMTAgMTUwIi8+CiAgPHBhdGggY2xhc3M9ImZsb3dkIiBkPSJNOTY0IDE5NiBDIDcyMCAzMDAsIDU2MCAyMjAsIDQ4MCAxNzAiLz4KICA8dGV4dCB4PSI2MDAiIHk9IjI1MCIgY2xhc3M9InNtYWxsIiBmaWxsPSIjZTA3YjAwIj5PQU06IHNoYXJlIG1ldHJpY3Mg4oaSIGNlbnRyYWwgU2luazwvdGV4dD4KICA8cGF0aCBjbGFzcz0iZmxvd2QiIGQ9Ik04MDAgMzIwIEMgNjAwIDMyMCwgNTQwIDI1MCwgNDgwIDIwMCIvPgogIDxwYXRoIGNsYXNzPSJmbG93IiBkPSJNMjMyIDI0MyBMIDI1MCAyNDMiLz4KICA8cGF0aCBjbGFzcz0iZmxvdyIgZD0iTTMwMCAxOTAgTCAxNjAgMjEwIi8+CgogIDx0ZXh0IHg9IjMwIiB5PSI0MTAiIGNsYXNzPSJzdWIiIGZvbnQtd2VpZ2h0PSI3MDAiPkxlZ2VuZDo8L3RleHQ+CiAgPGxpbmUgeDE9IjkwIiB5MT0iNDA1IiB4Mj0iMTMwIiB5Mj0iNDA1IiBjbGFzcz0iZmxvdyIvPgogIDx0ZXh0IHg9IjEzNiIgeT0iNDA5IiBjbGFzcz0ic21hbGwiPmNvbnRyb2wgLyBxdWVyeSAvIGFsZXJ0IHBhdGg8L3RleHQ+CiAgPGxpbmUgeDE9IjMwMCIgeTE9IjQwNSIgeDI9IjM0MCIgeTI9IjQwNSIgY2xhc3M9ImZsb3dkIi8+CiAgPHRleHQgeD0iMzQ2IiB5PSI0MDkiIGNsYXNzPSJzbWFsbCI+Y3Jvc3MtYWNjb3VudCBtZXRyaWMgYWdncmVnYXRpb24gKE9BTSk8L3RleHQ+CgogIDxyZWN0IHg9IjMwIiB5PSI0MzAiIHdpZHRoPSIxMTIwIiBoZWlnaHQ9IjEyMCIgcng9IjEwIiBmaWxsPSIjZmFmYWZhIiBzdHJva2U9IiNjY2MiLz4KICA8dGV4dCB4PSI0NiIgeT0iNDU0IiBjbGFzcz0ibGJsIiBmb250LXdlaWdodD0iNzAwIj5EYXRhIGNvbGxlY3Rpb24gKHR3byBjb21wbGVtZW50YXJ5IHBhdGhzKTwvdGV4dD4KICA8dGV4dCB4PSI0NiIgeT0iNDc2IiBjbGFzcz0ic21hbGwiPjEuIENvbnRpbnVvdXMgKHByaW1hcnkpOiBDbG91ZFdhdGNoIEFnZW50IOKGkiBkaXNrX3VzZWRfcGVyY2VudCwgaW5vZGVzX2ZyZWUsIHVzZWQsIHRvdGFsIGV2ZXJ5IDYwcywgZGltZW5zaW9uZWQgYnkgSW5zdGFuY2VJZCArIE1vdW50UGF0aC48L3RleHQ+CiAgPHRleHQgeD0iNDYiIHk9IjQ5NiIgY2xhc3M9InNtYWxsIj4yLiBPbi1kZW1hbmQgLyBmYWxsYmFjazogQW5zaWJsZSBtb25pdG9yaW5nIHJvbGUgcnVucyBgZGZgIG92ZXIgU1NNIOKGkiBwdXQtbWV0cmljLWRhdGEgdG8gdGhlIEN1c3RvbURpc2sgbmFtZXNwYWNlIChhbHNvIHVzYWJsZSB2aWEgY3JvbiBzY3JpcHQpLjwvdGV4dD4KICA8dGV4dCB4PSI0NiIgeT0iNTIwIiBjbGFzcz0ic21hbGwiPkFnZ3JlZ2F0aW9uOiBPQU0gbWFrZXMgYWxsIG1lbWJlci1hY2NvdW50IG1ldHJpY3MgcXVlcnlhYmxlIGluIHRoZSBtb25pdG9yaW5nIGFjY291bnQg4oCUIG9uZSBkYXNoYm9hcmQsIG9uZSBzZXQgb2YgYWxhcm1zLCBubyBFVEwgLyBkYXRhIG1vdmVtZW50LjwvdGV4dD4KICA8dGV4dCB4PSI0NiIgeT0iNTQwIiBjbGFzcz0ic21hbGwiPlNjYWxlOiBhd3NfZWMyIHRhZyBkaXNjb3ZlcnkgKyBTdGFja1NldCBvcmcgYXV0by1kZXBsb3ltZW50ICsgU1NNIFN0YXRlIE1hbmFnZXIgZHJpZnQgY29ycmVjdGlvbiA9IHplcm8gbWFudWFsIHBlci1WTSB3b3JrLjwvdGV4dD4KCiAgPHRleHQgeD0iMzAiIHk9IjU4OCIgY2xhc3M9ImxibCIgZm9udC13ZWlnaHQ9IjcwMCI+RW5yb2xsbWVudCBsaWZlY3ljbGU8L3RleHQ+CiAgPHJlY3QgeD0iMzAiIHk9IjYwMCIgd2lkdGg9IjExMjAiIGhlaWdodD0iNjAiIHJ4PSIxMCIgZmlsbD0iI2VlZjRmZiIgc3Ryb2tlPSIjM2I2ZWE1Ii8+CiAgPHRleHQgeD0iNDYiIHk9IjYyNCIgY2xhc3M9InNtYWxsIj5MYXVuY2ggdGFnZ2VkIFZNIChNb25pdG9yaW5nPUVuYWJsZWQpICDihpIgIGF3c19lYzIgaW52ZW50b3J5IGRpc2NvdmVycyBpdCAg4oaSICBjbG91ZHdhdGNoX2NvbmZpZyByb2xlIGluc3RhbGxzL2NvbmZpZ3VyZXMgYWdlbnQgb3ZlciBTU008L3RleHQ+CiAgPHRleHQgeD0iNDYiIHk9IjY0NCIgY2xhc3M9InNtYWxsIj7ihpIgIG1ldHJpY3MgZmxvdyB0byBsb2NhbCBDbG91ZFdhdGNoICDihpIgIE9BTSBMaW5rIHNoYXJlcyB0byBjZW50cmFsIFNpbmsgIOKGkiAgZGFzaGJvYXJkcyArIGFsYXJtICg4MCUpIOKGkiBTTlMuICBBZGRpbmcgYW4gYWNjb3VudDogU3RhY2tTZXQgYXV0by1kZXBsb3lzIGV2ZXJ5dGhpbmcuPC90ZXh0Pgo8L3N2Zz4=)

*A simplified view is in docs/architecture-simple.svg.*

```
                    +--------------------------------+
                    |      AWS Organizations         |
                    | (Multiple AWS Accounts)        |
                    +---------------+----------------+
                                    |
                           Cross-Account IAM Role
                                    |
                    +---------------v----------------+
                    |      Monitoring Account        |
                    | Ansible Control Node           |
                    | CloudWatch Dashboard + Alarms  |
                    | SNS Notifications              |
                    +---------------+----------------+
                                    |
                        AWS Systems Manager (SSM)
                                    |
        -------------------------------------------------------
        |                      |                      |
+---------------+     +---------------+     +---------------+
| AWS Account A |     | AWS Account B |     | AWS Account C |
| EC2 + SSM     |     | EC2 + SSM     |     | EC2 + SSM     |
| CloudWatch    |     | CloudWatch    |     | CloudWatch    |
| Agent         |     | Agent         |     | Agent         |
+-------+-------+     +-------+-------+     +-------+-------+
        +---------------------+---------------------+
                              |
                 CloudWatch Metrics (via OAM)
                              |
                   Central Dashboard & Alerts

```

---

## 2. Repository Structure

```
disk-monitoring/
├── ansible.cfg
├── requirements.yml
├── inventories/
│   └── aws_ec2.yml                 # EC2 dynamic inventory (tag-based discovery)
├── group_vars/
│   └── all.yml                     # SSM connection + thresholds (single source of truth)
├── playbooks/
│   ├── discover_instances.yml      # validate SSM reachability
│   ├── install_agent.yml           # install CloudWatch Agent
│   ├── configure_agent.yml         # deploy config + start agent
│   ├── verify_agent.yml            # verify + on-demand df snapshot
│   └── deploy.yml                  # runs all of the above, in order
├── roles/
│   ├── discovery/                  # SSM connectivity check
│   ├── cloudwatch_install/         # OS-aware agent install (from official AWS URL)
│   ├── cloudwatch_config/          # templated config + restart-on-change handler
│   └── monitoring/                 # verify agent + df fallback collector
├── templates/
│   └── cloudwatch-agent.json.j2    # readable copy of the agent config
├── cloudformation/
│   ├── cross-account-role.yaml         # access mgmt: role the hub assumes (StackSet)
│   ├── ec2-instance-profile.yaml       # access mgmt: badge on each VM (SSM + CW)
│   ├── monitoring-account-oam-sink.yaml# aggregation: central metrics sink
│   ├── member-account-oam-link.yaml    # aggregation: per-account link -> sink (StackSet)
│   └── disk-alarms.yaml                # alerting: SNS + single 80% disk-usage alarm
└── docs/
    ├── architecture.svg
    └── architecture-simple.svg

```

---

## 3. The Four Requirements → Where They Live

### A. Data Collection

- **Primary:** `roles/cloudwatch_install` + `roles/cloudwatch_config` install and configure the CloudWatch Agent, which emits `disk_used_percent`, inodes, used, total **every 60s**, dimensioned by `InstanceId` + mount path.
- **Fallback / on-demand:** `roles/monitoring` runs `df` over SSM and publishes a custom metric.

### B. Data Aggregation

- **CloudWatch cross-account Observability (OAM):** each account creates a **Link** (`member-account-oam-link.yaml`) to a single **Sink** (`monitoring-account-oam-sink.yaml`) in the monitoring account → one dashboard, one set of alarms, **no ETL**.
- **Alerting:** `disk-alarms.yaml` — SNS topic + a single **80%** disk-usage alarm.

### C. Access Management

- **No SSH.** `group_vars/all.yml` sets `ansible_connection: aws_ssm`, so Ansible reaches every VM through **AWS Systems Manager** — no keys, no bastion, no port 22, fully audited via CloudTrail.
- **Cross-account:** the hub uses **STS AssumeRole** into `DiskMonitoringCrossAccountRole` (`cross-account-role.yaml`, least-privilege, `ExternalId`-scoped). VMs carry an instance profile (`ec2-instance-profile.yaml`) granting only SSM + CloudWatch agent permissions.

### D. Scalability

- **Discovery is automatic:** `inventories/aws_ec2.yml` returns every running VM tagged `Monitoring=Enabled` — no manual host lists.
- **Enrollment is org-wide & self-healing:** CloudFormation **StackSets** roll the IAM role, instance profile, and OAM link to every account (incl. new accounts via Organizations auto-deployment); **SSM State Manager** keeps the agent installed under drift.

---

## 4. Quick Start

### Prerequisites (on the Ansible control node)

```bash
pip install "ansible>=9" boto3 botocore
ansible-galaxy collection install -r requirements.yml
# Install the Session Manager plugin for the aws_ssm connection.

```

Set `ansible_aws_ssm_bucket_name` in `group_vars/all.yml` to an S3 bucket you own.

### Step 1 — Access management (deploy once per member account, via StackSet)

```bash
aws cloudformation deploy --template-file cloudformation/ec2-instance-profile.yaml \
  --stack-name disk-mon-instance-profile --capabilities CAPABILITY_NAMED_IAM

aws cloudformation deploy --template-file cloudformation/cross-account-role.yaml \
  --stack-name disk-mon-xacct-role --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides MonitoringAccountId=111122223333

```

### Step 2 — Enroll all VMs in an account (one command)

```bash
AWS_PROFILE=member-acct-A ansible-playbook playbooks/deploy.yml
# discover -> install -> configure -> verify, across every tagged VM in parallel

```

### Step 3 — Central aggregation

```bash
# In the MONITORING account:
aws cloudformation deploy --template-file cloudformation/monitoring-account-oam-sink.yaml \
  --stack-name disk-mon-oam-sink --parameter-overrides OrgId=o-xxxxxxxxxx
# In EACH member account (StackSet):
aws cloudformation deploy --template-file cloudformation/member-account-oam-link.yaml \
  --stack-name disk-mon-oam-link \
  --parameter-overrides SinkArn=arn:aws:oam:us-east-1:111122223333:sink/xxxx

```

### Step 4 — Alerting

```bash
aws cloudformation deploy --template-file cloudformation/disk-alarms.yaml \
  --stack-name disk-mon-alarms \
  --parameter-overrides NotificationEmail=oncall@example.com

```

---

## 5. Solution Workflow

```
Developer launches EC2  →  tag Monitoring=Enabled  →  aws_ec2 inventory discovers it
  →  ansible-playbook deploy.yml (discover → install → configure → verify)
  →  CloudWatch Agent collects disk metrics (60s)  →  metrics flow via OAM to the hub
  →  central dashboard updates  →  alarm at 80%  →  SNS → email / Slack

```

---

## 6. Key Components Summary

| Requirement | Solution |
| --- | --- |
| Data Collection | CloudWatch Agent installed & configured by Ansible (df fallback via `monitoring` role) |
| Aggregation | Central CloudWatch dashboard fed by cross-account **OAM** |
| Access Management | **IAM roles + AWS Systems Manager (SSM)**, no SSH keys, audited by CloudTrail |
| VM Discovery | **EC2 Dynamic Inventory** plugin (tag `Monitoring=Enabled`) |
| Enrollment | Automatic via Ansible playbooks/roles over SSM |
| Scalability | New EC2s auto-discovered & configured; StackSets seed new accounts |
| Alerting | CloudWatch Alarm (80%) + Amazon SNS |
| Security | IAM, SSM, CloudTrail, least privilege, no inbound ports |

### Access Management (detail)

| Component | Purpose |
| --- | --- |
| IAM Role | Secure, least-privilege permissions for EC2 and Ansible |
| AWS Systems Manager | Secure access without SSH |
| Cross-Account IAM Role | Access multiple AWS accounts from one hub |
| CloudTrail | Audit all actions |

### VM Discovery & Enrollment (detail)

- **Discovery:** Ansible queries the AWS EC2 API via the dynamic inventory plugin; any running instance tagged `Monitoring=Enabled` appears automatically — no IPs to maintain.
- **Enrollment:** on discovery, Ansible (over SSM) → installs the agent → configures monitoring → starts it → verifies status. The server then streams disk metrics to CloudWatch.

