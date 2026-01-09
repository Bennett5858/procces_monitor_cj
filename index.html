"""
Processs monitor script that watches a specified process by name or PID, and restarts it if it exceeds defined memory or CPU usage thresholds.

"""

import argparse
import time
import psutil
import subprocess
import logging
from datetime import datetime

try:
    import requests
except Exception:
    requests = None  # optional

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s [%(levelname)s] %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

def find_processes_by_name(name):
    procs = []
    for p in psutil.process_iter(['pid', 'name', 'cmdline']):
        try:
            if p.info['name'] == name or (p.info['cmdline'] and name in ' '.join(p.info['cmdline'])):
                procs.append(p)
        except (psutil.NoSuchProcess, psutil.AccessDenied):
            continue
    return procs

def notify_webhook(webhook_url, message):
    if not webhook_url:
        return
    if requests is None:
        logging.warning("requests not installed — пропускаем уведомление")
        return
    try:
        requests.post(webhook_url, json={"text": message}, timeout=5)
    except Exception as e:
        logging.exception("Не удалось отправить webhook: %s", e)

def restart_via_systemctl(service_name):
    try:
        subprocess.run(["systemctl", "restart", service_name], check=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        return True, "systemctl restart выполнен"
    except Exception as e:
        return False, f"systemctl restart failed: {e}"

def restart_via_command(cmd):
    try:
        subprocess.run(cmd, shell=True, check=True)
        return True, f"Команда рестарта `{cmd}` выполнена"
    except Exception as e:
        return False, f"Команда рестарта `{cmd}` не удалась: {e}"

def kill_and_start(proc, start_cmd=None):
    try:
        logging.info("Завершаем PID %s (%s)", proc.pid, proc.name())
        proc.terminate()
        try:
            proc.wait(timeout=10)
        except psutil.TimeoutExpired:
            logging.warning("Процесс не завершился — принудительно убиваем")
            proc.kill()
        if start_cmd:
            logging.info("Запускаем: %s", start_cmd)
            subprocess.Popen(start_cmd, shell=True)
        return True, "kill+start выполнены"
    except Exception as e:
        logging.exception("Ошибка при kill/start: %s", e)
        return False, str(e)

def main(args):
    logging.info("Запуск ProcessWatchdog для '%s' (интервал=%ss)", args.name, args.interval)
    last_action = None
    while True:
        procs = []
        if args.pid:
            try:
                p = psutil.Process(int(args.pid))
                procs = [p]
            except Exception:
                procs = []
        else:
            procs = find_processes_by_name(args.name)

        if not procs:
            logging.warning("Процесс не найден: %s", args.name if not args.pid else f"pid {args.pid}")
            # опционально попытаться запустить
            if args.start_cmd and (not last_action or (time.time() - last_action) > args.cooldown):
                ok, msg = restart_via_command(args.start_cmd)
                logging.info("Попытка старта: %s, %s", ok, msg)
                if ok:
                    notify_webhook(args.webhook, f"[Watchdog] {args.name}: started at {datetime.now()}.")
                    last_action = time.time()
        else:
            # проверяем каждый процесс
            for p in procs:
                try:
                    mem_mb = p.memory_info().rss / 1024**2
                    cpu = p.cpu_percent(interval=0.1)  # короткий measurement
                    logging.debug("PID %s mem=%.1fMB cpu=%.1f%%", p.pid, mem_mb, cpu)
                    if (args.max_mem_mb and mem_mb > args.max_mem_mb) or (args.max_cpu and cpu > args.max_cpu):
                        logging.warning("Триггер для PID %s: mem=%.1fMB cpu=%.1f%%", p.pid, mem_mb, cpu)
                        # предпринимаем шаги: systemctl -> start_cmd -> kill+start
                        action_taken = False
                        if args.service and (not last_action or (time.time() - last_action) > args.cooldown):
                            ok, msg = restart_via_systemctl(args.service)
                            logging.info("systemctl restart: %s, %s", ok, msg)
                            action_taken = ok
                        if not action_taken and args.start_cmd and (not last_action or (time.time() - last_action) > args.cooldown):
                            ok, msg = restart_via_command(args.start_cmd)
                            logging.info("start_cmd: %s, %s", ok, msg)
                            action_taken = ok
                        if not action_taken and (not last_action or (time.time() - last_action) > args.cooldown):
                            ok, msg = kill_and_start(p, start_cmd=args.start_cmd)
                            logging.info("kill/start: %s, %s", ok, msg)
                            action_taken = ok
                        notify_webhook(args.webhook, f"[Watchdog] {args.name} ({p.pid}) restarted at {datetime.now()}. Reason: mem={mem_mb:.1f}MB cpu={cpu:.1f}%")
                        last_action = time.time()
                except (psutil.NoSuchProcess, psutil.AccessDenied):
                    continue

        time.sleep(args.interval)

        if __name__ == "__main__":
            parser = argparse.ArgumentParser(description="ProcessWatchdog — monitor and restart a process if resource usage exceeds thresholds")
            parser.add_argument("--name", help="Process name or part of cmdline (e.g., my_service)", default=None)
            parser.add_argument("--pid", help="Process PID (takes priority over --name)", default=None)
            parser.add_argument("--max-mem-mb", type=float, help="Memory threshold in MB", default=1024.0)
            parser.add_argument("--max-cpu", type=float, help="CPU threshold (%)", default=80.0)
            parser.add_argument("--interval", type=int, help="Check interval in seconds", default=15)
            parser.add_argument("--service", help="If service is managed by systemd — service name for systemctl restart", default=None)
            parser.add_argument("--start-cmd", help="Command to start the process (shell string)", default=None)
            parser.add_argument("--webhook", help="Webhook URL for notifications (optional)", default=None)
            parser.add_argument("--cooldown", type=int, help="Minimum seconds between restarts", default=300)
            args = parser.parse_args()
            if not args.name and not args.pid:
                parser.error("Must specify --name or --pid")
            main(args)
