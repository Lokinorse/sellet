<script>
    import { HOST } from './config';
    import { formToJSON, getFormErrors, setForm, getForm, makeFormData, formFromDataAttributes } from './helpers/formHelpers'
    import { reachGoals, getKeys } from './helpers/helpers'
    import Loader from './components/Loader.svelte'
    import Modal from './components/Modal.svelte'
    import Refill from './components/Refill.svelte'
    import ErrorsRender from './components/ErrorsRender.svelte'
    import Thanks from './components/Thanks.svelte'

    let color;
    let state = null;
    let errors = {};

    let form = {
        name: '',
        phone: '',
        selltype: '',
        cost: '',
        offer: '',
        title: '',
        agree: false
    };

    document.addEventListener('submit', function (event) {
        if (!event.target.tagName.toLowerCase() === 'form') return;
        event.preventDefault();
        event.stopPropagation();
        form = formToJSON(this);
        handleLandingFormSubmit(setForm(form))
    }, false);

    document.addEventListener('click', function (event) {
        if (!event.target.classList.contains('sellet-buy-button')) return;
        event.preventDefault();
        event.stopPropagation();
        form = getForm(form)
        form = Object.assign({}, form, formFromDataAttributes(event.target));
        state = 'refill';
    }, false);

    async function handleLandingFormSubmit(form)  {
        const validationErrors = getFormErrors(form);
        if (getKeys(validationErrors).length) {
            state = 'refill';
            errors = validationErrors;
            return
        }
        state = 'loading';
        reachGoals('sell-success')
        const res = await fetch(`${HOST}/order.php`, {
            method: 'POST',
            body: makeFormData(form)
        });
        state = 'thanks';
    }

    function handleCloseModal () {
        state = null;
    }

    function onRefill() {
        handleLandingFormSubmit(form)
        makeRedirect(form)

    }

    function makeRedirect(form) {
        if (!formData.redirect) return
        setTimeout(function() {
            window.location = form.redirect
        }, 1000)
    }   
    
    

</script>

{#if state === 'refill'}
    <Modal header="Форма заявки" onClose={handleCloseModal}>
        <Refill onRefill={onRefill} form={form} errors={errors} />
    </Modal>
{/if}

{#if state === 'errors' && errors.length}
    <Modal header="Ошибка!" onClose={handleCloseModal}>
        <ErrorsRender errors={errors} />
    </Modal>
{/if}

{#if state === 'loading'}
    <Modal header="Подождите" onClose={handleCloseModal}>
        <Loader/>
    </Modal>
{/if}

{#if state === 'thanks'}
    <Modal header="Спасибо за заказ" onClose={handleCloseModal}>
        <Thanks/>
    </Modal>
{/if}
