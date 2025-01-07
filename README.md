# Kubernetes-Web-App
## Задание
В этом репозитории представлен yaml-манифест для решения следующего задания

Опишите решение для веб-приложения в kubernetes в виде yaml-манифеста. Оставляйте в коде комментарии по принятым решениям. Есть следующие вводные:

- у нас мультизональный кластер (три зоны), в котором пять нод
- приложение требует около 5-10 секунд для инициализации
- по результатам нагрузочного теста известно, что 4 пода справляются с пиковой нагрузкой
на первые запросы приложению требуется значительно больше ресурсов CPU, в дальнейшем потребление ровное в районе 0.1 CPU. По памяти всегда “ровно” в районе 128M memory
- приложение имеет дневной цикл по нагрузке – ночью запросов на порядки меньше, пик – днём
- хотим максимально отказоустойчивый deployment
- хотим минимального потребления ресурсов от этого deployment’а

## Пояснения по компонентам манифеста

### Deployment

- **replicas**: Равно 4 для обработки пиковой нагрузки.
- **resources**:
  - `requests`: минимальный запрос ресурсов для равномерного распределения нагрузки.
  - `limits`: ограничение на дополнительные ресурсы для обработки первых запросов, требующих больших вычислений.
- **readinessProbe**:
  - Проверяет готовность контейнера.
  - Установлено `initialDelaySeconds: 5`, так как приложение инициализируется за 5–10 секунд.
- **livenessProbe**:
  - Перезапускает зависший контейнер.
  - `initialDelaySeconds: 10` учитывает время запуска.
- **affinity**:
  - Гарантирует равномерное распределение подов по зонам для увеличения отказоустойчивости.

### HorizontalPodAutoscaler

- **Масштабирование на основе CPU**:
  - Минимум 2 реплики для ночного времени.
  - Максимум 8 для дневного пика.
  - Целевая загрузка CPU составляет 50% для сбалансированного использования ресурсов.

### ClusterAutoscaler

- **Настройки оптимизации кластера**:
  - Период сканирования установлен на 10 секунд для своевременного реагирования на изменения.
  - Увеличение или уменьшение числа нод выполняется с минимальной задержкой.
  - Распределение нагрузки между нодами для минимизации простоев.

## Для развёртывания необходимо

1. Применить манифесты:
   ```bash
   kubectl apply -f Deployment.yaml
   kubectl apply -f HorizontalPodAutoscaler.yaml
   kubectl apply -f ClusterAutoscaler.yaml
   ```
2. Проверить распределение под:
   ```bash
   kubectl get pods -o wide
   ```
3. Проверить автоскейлинг:
   ```bash
   kubectl get hpa
   ```

