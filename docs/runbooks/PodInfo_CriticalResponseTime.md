Алерт: PodInfo_CriticalResponseTime
Severity: CRIT
Impact: Podinfo: КРИТИЧЕСКОЕ время ответа > 5с, пользователи массово получают ошибки таймаута или долгих зависаний в работе сервиса

1. Подтвердите проблему
    Откройте дашборд времени ответа сервиса PodInfo и убедитесь, что показатели rate(http_request_duration_seconds_sum{job="podinfo"}[1m]) / rate(http_request_duration_seconds_count{job="podinfo"}[1m]) действительно превышают допустимый порог времени ответа.
    Откройте дашборд Kubernetes / Compute Resources / Namespace (Pods), выберите переменную namespace demo-runbook и убедитесь, что ошибка возникает в связи с увеличением нагрузки на систему (CPU, MEM, IO), а не из-за общего сбоя сети или балансировщика.
    Посмотри логи поды
    kubectl logs -n demo-runbook -l app.kubernetes.io/name=podinfo --tail=100
    Создайте статус инцидента

2. Если проблема связана с резким ростом нагрузки CPU/MEM следует увеличить количество реплик сервиса пропорционально возросшей нагрузке
    Подключитесь к кластеру и добавьте больше реплик сервису
    kubectl scale deployment podinfo --replicas=3 -n demo-runbook. Запишите это в статусе инцидента

    Проверьте, что новые реплики успешно инициировались
    kubectl get pods -n demo-runbook -l app=podinfo

3. Если ошибка не исправлена и проблема связана с вводом нового релиза сервиса podinfo следует откатить helm релиз
    Посмотрите историю релизов (в дашборде или в консоли helm history {название релиза} -n demo-runbook)
    Производим откат сервиса на -1 релиз
    helm rollback {название релиза} -n demo-runbook . Запишите это в статусе инцидента

    Проверьте, что поды с предыдущим релизом инициировались
    kubectl get pods -n demo-runbook -l app=podinfo

4. Если ошибка связана с зависанием работающих процессов, попробуй рестарт под
    kubectl rollout restart deployment podinfo -n demo-runbook. Запишите это в статусе инцидента

5. Проверьте, что пользователям стало лучше
    На дашборде времени ответа, метрики rate(http_request_duration_seconds_sum{job="podinfo"}[1m]) / rate(http_request_duration_seconds_count{job="podinfo"}[1m]) должны заметно упасть
    На дашборде нагрузки Ops на бизнес процессы PodInfo метрики зеленые и соответствуют SLA
    На дашборде Kubernetes / Compute Resources / Namespace (Pods) утилизация ресурсов стала меньше, нет деградаций системных показателей нагрузки

5. Закройте инцидент и оставьте след
    В статусе инцидента зафиксируйте время переключения, что помогло, что делали. Создайте задачу: «проверить причину CriticalResponseTime и обновить playbook/мониторинг»
