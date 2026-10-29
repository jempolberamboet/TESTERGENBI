select p.e, p.n, s.n as sesi, to_timestamp(a.t/1000) at time zone 'Asia/Jakarta' as waktu, a.dv
from att a join people p on p.id = a.pid join sessions s on s.id = a.sid
where p.e like 'tester%' order by a.t;
