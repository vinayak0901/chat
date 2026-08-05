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

Subject
Clarification on Production Environment Behavior
Hi Team,
I recently received the update regarding the "Fix logout with platform lock" issue.
Could you please clarify whether the same behavior or situation is expected in the production environment as well? I would like to confirm if this fix applies to production in the same way or if there are any differences we should be aware of.
Thank you for your clarification.
