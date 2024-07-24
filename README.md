def modified_retry(max_retries, timeout, exception=None, error_msg=None, period=1):
    def outer(func):
        def inner(*args, **kwargs):
            end_time = time.time() + timeout
            retries = max_retries
            while retries:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if (e and e == exception) or (error_msg and error_msg in str(e)):
                        log.error(f'ERROR: {e}')
                        if time.time() > end_time:
                            log.error(f'Failed to successfully execute cmd during {timeout} sec')
                            raise e
                        if retries == 1:
                            log.error(f'Failed to successfully execute cmd with {max_retries} attempts')
                            raise e
                        retries -= 1
                        time.sleep(period)
                    else:
                        raise e
        return inner
    return outer

# Будем делать повторные попытки, только если упали с TimeoutExpired
@retry(5, 60, exception=subprocess.TimeoutExpired)
def send_request():
    raise TimeoutExpired('Operation timed out. This is expected.')

# Тест
send_request()
