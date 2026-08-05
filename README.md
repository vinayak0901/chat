# chat
((ROUND(G31,4)-TRUNC(ROUND(G31,4)))*10000)+(25-MOD(((ROUND(G31,4)-TRUNC(ROUND(G31,4)))*10000),25)) is giving 9025 G31 is 109.900015

=((ROUND(G31,4)-TRUNC(ROUND(G31,4)))*10000)
+MOD(25-MOD(((ROUND(G31,4)-TRUNC(ROUND(G31,4)))*10000),25),25

=ROUND(((ROUND(G31,4)-TRUNC(ROUND(G31,4)))*10000,0)
+MOD(25-MOD(ROUND(((ROUND(G31,4)-TRUNC(ROUND(G31,4)))*10000,0),25),25)

=ROUND((ROUND(G31,4)-TRUNC(ROUND(G31,4)))*10000,0)
+MOD(
25-MOD(ROUND((ROUND(G31,4)-TRUNC(ROUND(G31,4)))*10000,0),25),
25
)

We recently received the FIX Logout message with text "System is not open for business" while we were connected to the SEP Market Taker API.
Immediately after try to reconnect by sending the FIX Logon message - we were receiving Logout with text "Platform is locked for external users"
Could you please clarify whether the same behavior or situation is expected in the production environment as well? I would like to confirm if this fix applies to production in the same way or if there are any differences we should be aware of.
Thank you for your clarification.
